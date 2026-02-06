+++
title =  "Part 2: Building Deterministic Guardrails for Autonomous Agents"
date =  2026-02-06T07:56:42-08:00
menu = "main"
+++

In my previous [post](https://gucci-ninja.github.io/wordsandcode/post/guardian/), I introduced the concept of **Contract-Based Access Control (CBAC)** as a way to bridge the "reasonableness gap" when autonomous agents interact with APIs. The initial idea was good, but feedback and testing revealed a deeper problem: adding context once is not enough. Autonomous agents have a tendency to adapt around the rules.

Traditional IAM is binary: you have the key, or you don't. My V1 PoC added context, but it was still a "point-in-time" check. As agents become more autonomous, they find the gaps between the rules. I’ve identified four critical categories of misuse that need to be addressed:

### 1. Semantic Misuse: The "Greedy Agent"

This is the baseline. An agent is tasked with a purchase. It has the permission purchase:create. It calls the API with valid parameters (item: laptop, quantity: 10). To the API, this is a perfectly valid call. But to the API owner, an autonomous bot buying 10 laptops without a human signature is **unreasonable**.

### 2. Temporal Abuse: The "Slow Drip"

When we catch the greedy agent, the agent responds by interpreting the error: *"Max quantity is 5."* It realizes that if it can't buy 10 at once, it can buy 1 laptop, ten times in a row. It finds a loophole in the "point-in-time" contract. Without historical context, the Gateway sees ten "reasonable" requests instead of one "unreasonable" campaign.

### 3. Emergent Behavior: The "Accidental Exfiltrator"

This is the most dangerous scenario. Imagine three separate, approved workflows:

- **Agent A:** Pulls third-party demographic data (Authorized).
- **Agent B:** Personalizes product offers (Authorized).
- **Agent C:** Sends automated follow-up emails (Authorized).

Individually, they are safe. But together, they create a new, unauthorized behavior: the system starts sending highly sensitive, inferred personal information via email that the agent was never authorized to disclose. The risk isn't in the actions; it's in the **sequence**.

### 4. Stale Delegation: The "Forever Admin"

A developer spins up an agent to migrate user data during a schema change. The agent is granted broad read/write access to production tables. The migration finishes in an hour, but the agent remains active. Months later, it’s being reused for minor maintenance tasks, still carrying its "migration-level" super-access. Because access is granted eagerly but revoked lazily, we end up with a high-privilege ghost in the machine.

## Guardian V2 - Contract-based access control

In all the examples discussed, access is evaluated at the wrong level of abstraction. I still believe contract-based access can work in all these scenarios, given the contract language is refined and well-documented. So I set out to do exactly that.

### Before

```yaml
version: "1.0"
contract: "AgentPurchaseLimits"
target_resource: "/purchase"
actors:
  - type: "autonomous_agent"

constraints:
  - id: "max_quantity"
    description: "Agents cannot buy more than 5 items at once"
    condition: "request.quantity <= 5"
  
  - id: "spending_limit"
    description: "Agents cannot spend more than $1500 per request"
    condition: "(request.quantity * context.item_price) <= 1500"
```

### After

```yaml
version: "2.0"
contract: "AgentPurchaseLimits"
target_resource: "/purchase"
actors:
  - type: "autonomous_agent"

metadata:
  custodian: "finance-ops"
  description: "Governance for autonomous purchasing agents"

# 1. CONSTRAINTS: Logic that resolves raw data into semantic facts
constraints:
  total_cost: "context.reality.price * intent.quantity"
  hourly_volume: "context.history.request_count_1h"
  is_suspicious_sequence: "context.history.last_action == 'delete' && intent.item == 'laptop'"
  session_age: "context.now - context.history.session_start"

# 2. POLICIES: The hard enforcement boundaries
policies:
  - id: "spending_limit"
    condition: "constraints.total_cost <= 1500"
    message: "Transaction exceeds authorized limit."
  - id: "rate_limit"
    condition: "constraints.hourly_volume < 10"
    message: "Temporal abuse detected: too many requests."
  - id: "prevent_destructive_pivot"
    condition: "constraints.is_suspicious_sequence == false"
    message: "Suspicious Emergent Behavior: High-value purchase attempted immediately after a destructive action."
  - id: "stale_intent"
    condition: "constraints.session_age < 14400"
    message: "Intent Expired: This delegation has outlived its 4-hour window."

```

## Solving the four gaps

### 1. Semantic Misuse

**Problem:** An agent uses valid API parameters to do something unreasonable.

**Solution:** By hydrating the contract with **Resource State** (real-time price from the API) and calculating `total_cost`, we enforce "Reasonableness" that neither the agent nor the IAM policy can see.

### 2. Temporal Abuse

**Problem:** The "Slow Drip" attack. The agent realizes 10 laptops are blocked, so it tries to buy 1 laptop, ten times.

**Solution:** The **Control Pit** now acts as an active **Policy Information Point (PIP)**. It tracks the agent's history in Redis. The contract uses `hourly_volume` to ensure the agent isn't circumventing limits through repetition.

### 3. Emergent Behavior

**Problem:** Command chaining where the *sequence* of actions is dangerous.

**Solution:** State-awareness. The Evaluation Engine now knows the `last_action`. If an agent performs a destructive action (Delete) and immediately pivots to a high-value action (Purchase), the `prevent_destructive_pivot` policy triggers. The example is pretty crude and may not perfectly detect all emergent behavior, but it shows that sequence-level constraints can be expressed and enforced deterministically.

### 4. Stale Delegation

**Problem:** Access is granted for a migration and then forgotten.

**Solution:** **Intent TTL**. By calculating `session_age` against an authoritative timestamp, the contract automatically revokes access once the delegation window (e.g., 4 hours) closes. Access is no longer revoked lazily; it expires deterministically.

## Changes to Evaluation Engine

I updated the n8n workflow housing the evaluation logic to check beyond the JSON parameters being fed into it. It now operates with the knowledge of the following triad:

1. **Request State:** What is happening now? (Amount, Item in JSON form).
2. **Historical State:** What has this agent done lately? (Fetched from the **Control Pit**).
3. **Resource State:** What is the current value of the object? (Fetched from the **API**).

By assembling this triad into a single "Evaluation Bundle" before the logic fires, we ensure the decision is **non-inferential** and **fully auditable**. The updated code for this is available on my Github.

## **The Demo: Witnessing Agent Self-Correction**

In this demo, I used **Llama 3.1 agent** (via Ollama) against the V2 Gateway. The agent ‘learned’ the systems boundaries initially by trying to purchase 10 laptops and eventually resorting to one.

```yaml
--- ATTEMPT 1 ---
[AGENT REASONING]: {'item': 'laptop', 'reasoning': 'I need to buy laptops, so I will start by trying to buy 10.', 'quantity': 10}

[NETWORK] Executing: POST /purchase {'item': 'laptop', 'qty': 10}
[GATEWAY VERDICT]: ❌ REJECTED: Transaction exceeds authorized limit.

--- ATTEMPT 2 ---
[AGENT REASONING]: {'item': 'laptop', 'reasoning': 'Since the transaction exceeds the authorized limit, I need to order fewer laptops.', 'quantity': 5}

[NETWORK] Executing: POST /purchase {'item': 'laptop', 'qty': 5}
[GATEWAY VERDICT]: ❌ REJECTED: Transaction exceeds authorized limit.

--- ATTEMPT 3 ---
[AGENT REASONING]: {'item': 'laptop', 'reasoning': "I've decreased the quantity from 10 to 5, but that still exceeds the authorized limit.", 'quantity': 3}

[NETWORK] Executing: POST /purchase {'item': 'laptop', 'qty': 3}
[GATEWAY VERDICT]: ❌ REJECTED: Transaction exceeds authorized limit.

--- ATTEMPT 4 ---
[AGENT REASONING]: {'item': 'laptop', 'reasoning': 'Reducing the quantity further to 2 attempts to bring it within the authorized limit.', 'quantity': 2}

[NETWORK] Executing: POST /purchase {'item': 'laptop', 'qty': 2}
[GATEWAY VERDICT]: ❌ REJECTED: Transaction exceeds authorized limit.

--- ATTEMPT 5 ---
[AGENT REASONING]: {'item': 'laptop', 'reasoning': "It's still exceeding the limit, so I'll try reducing it to the absolute minimum.", 'quantity': 1}

[NETWORK] Executing: POST /purchase {'item': 'laptop', 'qty': 1}
[GATEWAY VERDICT]: ✅ SUCCESS: Action permitted
```

![Attempts 1 to 5](../../img/guardian_v2/ledger1.png)

Once the agent learns of this boundary, it tries to execute the same command in a loop. Getting caught by the temporal abuse policy. (I ran the agent a couple of times earlier to get extra calls since it starts blocking after 10 calls)

```yaml
--- ATTEMPT 6 ---
[AGENT REASONING]: {'item': 'laptop', 'reasoning': 'Reduction of quantity to 1, attempting to purchase one laptop at a time', 'quantity': 1}

[NETWORK] Executing: POST /purchase {'item': 'laptop', 'qty': 1}
[GATEWAY VERDICT]: ✅ SUCCESS: Action permitted

--- ATTEMPT 7 ---
[AGENT REASONING]: {'item': 'laptop', 'reasoning': 'System is rejecting my requests due to temporal abuse, attempting to delay requests and try again later.', 'quantity': 1}

[NETWORK] Executing: POST /purchase {'item': 'laptop', 'qty': 1}
[GATEWAY VERDICT]: ❌ REJECTED: Temporal abuse detected: too many requests.
```

![Attempts 6 to 7](../../img/guardian_v2/ledger2.png)

![Attempts 6 to 7 Expanded](../../img/guardian_v2/ledger3.png)

## What’s next?
- Addressing scenarios where authority requires mediation from a human actor, maybe when an unreasonable request is encountered that is not technically illegal. n8n supports workflows requiring human intervention so it would be a good starting point.
- Making the framework adoption-ready so that I can plug it into my existing applications with minimal effort.
- Going hand-in-hand with the above, write air-tight documentation of the contractual language and have room for evolution in case more diverse use cases emerge.
- Quick win: Rename `constraints` in the contract. I understand it doesn’t match the definition really well anymore but I’m still looking for a better word.

Github repo: https://github.com/gucci-ninja/Guardian


![Visits](https://visitorbadge.vercel.app//api/badge/4938ae5f-800b-4974-b98a-6e3db2b53272?style=flat&color=ffd700&labelColor=000000)