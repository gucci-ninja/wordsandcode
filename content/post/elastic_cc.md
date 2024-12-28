+++
title =  "Elastic Community Conference Tech Talk"
date =  2021-04-02T22:17:26-05:00
menu = "main"
+++


**Developing a Decentralized Application with Elasticsearch and QR Codes**

---

### **Introduction**

Last year thanksgiving I wrote an article about technologies that I was grateful for and I wrote about 26 technologies starting from the letter A all the way to Z and some of these I had used before and some of them I had planned to use in the future. When I was looking for my next project to work on I thought of going back to that list and picking three at random to to sort of build something with. I ended up choosing Elasticsearch, QR codes, and blockchain for this project. That’s the backstory of how I got here.

### **Agenda**

In today’s talk, I’ll walk through my process of developing a decentralized application and how I integrated QR codes into it. Then, I’ll discuss how I leveraged Elasticsearch to observe transaction data on the blockchain and sync that data in real time. Finally, I’ll present a demo showcasing all the moving parts of the application, including the Elasticsearch dashboard.

## **Starting with an Idea**

When building something new, the process often begins with an idea or a problem you want to solve. From there, you identify the right technologies and stack to bring that idea to life.

However, I approached this project a bit differently. I had a set of tools I wanted to work with but no clear idea of what to build. So, I analyzed what each technology excels at:

- **Blockchain** is great for storing state and maintaining a reliable history of transactions.
- **QR codes** can serve as unique identifiers for tracking assets.
- **Elasticsearch** enables real-time data indexing and querying.

This led me to explore applications in the **supply chain industry**, where tracking shipments and ensuring data transparency are crucial. Blockchain’s immutability made it a strong fit for this use case.

But I wanted to take it a step further. Many supply chain applications already use distributed ledgers, but I wanted to incorporate the **consumer perspective**. Instead of just tracking an item from production to a distribution center, I wanted to follow its journey even after reaching the consumer.

This is especially relevant for farmed goods and local produce—allowing people to verify if something is truly locally sourced and when it was grown or harvested.

## **Building a Decentralized Application**

To build a decentralized application (dApp), you need four essential tools:

1. **A Smart Contract Language**
    - Smart contracts act as binding agreements that define the transactions on a blockchain.
    - I used **Solidity**, a language with a syntax similar to C++, to define the contract’s methods and data structures.
2. **Compilation and Migration Tools**
    - Once written, the smart contract needs to be compiled and deployed onto the blockchain.
    - I used **Truffle**, which simplifies compiling, migrating, and managing smart contracts.
    - Migration in blockchain is somewhat like database migrations, but with a key difference: when you migrate a smart contract, all previous transactions tied to the old contract become invalid because they no longer adhere to the new contract rules.
3. Deploy your contract
    - A local blockchain client running. For this, I chose **Ganache**, though **TestRPC** is another popular option.
    - Once your contract is written, compiled, and migrated, you can finally deploy it. Ganache is an Ethereum client that runs locally, meaning you don’t need to spend money on Ethereum gas fees while testing and compiling multiple times. It provides a space to host your contract, view transactions, and track mined blocks. We’ll take a look at it later in the demo.
4. Make transactions with Web3
    - Allows you to interact with the blockchain
    - Once your contract is deployed, you need a way to access its methods and retrieve data stored on the blockchain. Web3 provides a vast API for making transactions and interacting with smart contracts.
    - If you're building a dApp with **React Native**, an alternative to Web3 is **Drizzle**, which I also used since, to my knowledge, there isn’t a direct Web3 wrapper for React Native.

These four tools—Solidity, Truffle, Ganache, and Web3—were all I needed to build the dApp, and the best part was that it was completely free. Initially, I thought I’d need to invest money since Ethereum is involved, but I was able to develop and test everything without spending anything.

---

### **Smart Contract**

Now, moving on to the smart contract itself, I want to dive deeper into its contents.

I decided to store two main things:

1. **Items** – Each item has:
    - A unique **ID**
    - A **name**
    - A **tracking count** (number of times the item has been recorded in the system)
    - The **creator’s address** (who registered the item)
    - A **mapping of states**, which is essentially an array that tracks the item’s journey—from its initial registration to its current location.
2. **State** - Each state has
    - The date when state was recorded
    - The location where state was recorded
    - The **editor’s address** (who edited the item state)

From here, we’ll explore how these properties are used within the dApp.

### **QR Codes**

To enhance usability, I integrated **QR codes** into the system. Users can register items on the blockchain, and the application generates a QR code that can be printed and attached to the item. Every time someone scans the QR code, a new state is added to the blockchain, tracking the item’s journey over time.

### **Observibility with Elasticsearch**

Next, I’ll talk about observibility and how I used **Elasticsearch** to track transaction data.

To set up my Elasticsearch instance, I used **Docker Compose**, which I highly recommend for anyone working with Elastic products. It simplifies deployment—just configure the `docker-compose.yml` file, and you’re good to go. Within minutes, I had Elasticsearch up and running and was able to start building dashboards in **Kibana**.

---

### **Syncing Blockchain Data with Elasticsearch**

One of the biggest challenges was syncing my **blockchain** with **Elasticsearch**. There are two main scenarios for doing this:

1. **Existing Blockchain** – If the blockchain has already been running with transactions recorded, and you introduce Elasticsearch afterward.
2. **Simultaneous Startup** – If both the blockchain and Elasticsearch start at the same time, ensuring real-time synchronization.

I followed three key steps to achieve real-time syncing:

1. **Establishing Connections**
    - I set up a **Node.js** server to communicate with both **Ganache** (Ethereum client) and **Elasticsearch**, enabling them to interact seamlessly.
2. **Polling for New Blocks**
    - The server continuously monitors the blockchain for new blocks, ensuring that no transactions are missed.
3. **Extracting and Indexing Transactions**
    - Once a new block is found, the server checks if it contains transactions (such as item registrations or state updates).
    - If a transaction exists, it gets indexed into **Elasticsearch**, keeping it in sync with the blockchain.

---

### **Blockchain Code**

Here's a simplified version of how I iterated through the blockchain:

- **Start Block** = 0
- **Latest Block** = `getLatestBlock()` (Web3 API method)
- **Current Block** = Start Block

A **while loop** runs as long as `currentBlock < latestBlock`, processing each block and checking for transactions.

As new blocks are added, `latestBlock` updates dynamically, ensuring real-time synchronization.

---

### **Demo Environment**

I have four main components running:

1. **Elasticsearch & Kibana** (on the right)
    - Running via Docker Compose, providing real-time indexing and visualizations.
2. **Mobile App ("Digital Orange")** (mirrored from my phone)
    - Built with **React Native**, it integrates with Web3 for blockchain transactions and supports QR code generation/scanning.
3. **Ganache (Ethereum Client)** (bottom left)
    - Running the blockchain locally, displaying transactions and mined blocks.
4. **Node.js Server** (top left)
    - Polls the blockchain and syncs data with Elasticsearch.

At the moment, the server is continuously analyzing blocks, checking for new transactions. If no changes are detected, it skips them.

---

### **Registering an Item**

- Upon opening the app, the **login screen** appears (authentication via **Firebase**).
- I navigate to the **"Register Item"** section.
- I enter "Apples" as the item name and hit **Add**.
- The system generates a **QR code**, and a **blockchain transaction** is recorded.
- The transaction details (hash, gas used, block number) are displayed.
- The Node.js server attempts to index it in Elasticsearch.

At this point, indexing fails because the item only has **one recorded state**. To create **geo-based tracking**, we need at least **two states** to form a path.

---

### **Scanning an Item**

- I go to the **QR scanner** and scan an existing QR code.
- A new transaction is recorded, updating the item’s state.
- The transaction is successfully indexed in **Elasticsearch**.
- The timeline now shows timestamps and **latitude/longitude** coordinates for each recorded state.

*Note: The location data here is spoofed—I didn't actually travel from Brazil to the U.S. and then Antarctica!*

---

### **Dashboard**

There is a **dashboard** I built in **Kibana**, which visualizes blockchain data in Elasticsearch:

- **Map View** – Displays the journey of tracked items, with each color representing a unique item’s path.
- **Word Cloud** – Highlights the most frequently registered items.
- **Active Accounts** – Lists the most active blockchain addresses (Ganache provides 10 by default).

This visualization makes it easy to **track supply chains, verify authenticity, and ensure transparency**.

---

### **Final Thoughts**

Working with **blockchain, Elasticsearch, and QR codes** has been a **rewarding experience**. Initially, I found these technologies intimidating, but once I explored their documentation and built a project around them, I realized how powerful and accessible they are.

If you're new to **blockchain or Elasticsearch**, I hope you found some useful takeaways from my experience!

---

### **Resources**

Here are some useful links:

- **The DigitalOrange [Repo**](https://github.com/gucci-ninja/DigitalOrange) – Contains the full application and server-side code for syncing blockchain data to Elasticsearch.
- **Truffle Suite** – A great tool for Ethereum smart contract development.
- **Ether to Elasticsearch** – A project that inspired my approach to syncing blockchain data.

---

This was a talk I presented at the 2021 Elastic Community Conference. If you would like to watch the full thing please check out https://www.youtube.com/watch?v=XQbszbVQ3DE