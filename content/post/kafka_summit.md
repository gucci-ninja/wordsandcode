+++
title =  "Kafka Summit Tech Talk"
date =  2021-09-16T22:08:17-05:00
menu = "main"
+++

For those interested, I gave a talk at the Kafka APAC Summit in 2021 on how to leverage Kafka for decentralized applications. The video is still up on confluent’s website but I went and added most of the topics I discussed here. 

## What is a decentralized app (DApp)

Decentralized applications are programs that leverage a blockchain or peer-to-peer network to give users more control of their data. Decentralized applications are not really that new, you might have heard of some popular ones or even used them like BitTorrent. It uses the feeding and seeding model, where file sharing is made possible by users who volunteer to transfer their data to others and those users who propagate the data forwards by becoming seeders for the file.

In the case of blockchain, there’s a distributed ledger of blocks that are cryptographically hashed together. These blocks are filled with transactions that are governed by something called a smart contract. which is kind of like a binding agreement for your application. Usually there is also a front-end interface for users to make transactions for a small fee like if you’re on the Ethereum network that would be ether. For a number of reasons, decentralized applications are transforming the way the internet works.

## Popular use cases for DApps

Decentralized applications can be useful in industries where traceability and data integrity is very critical, like the supply chain or healthcare industry. Decentralized apps get rid of the need for a middle man in situations like voting day where you go cast your ballot and someone has to count it. There is also decentralized digital identity. For example, I recently learnend that they have started putting diplomas on a blockchain so they can’t be forged as easily. Which is cool, instead of an expensive piece of paper I get a block on a chain. Quite possibly the most popular form of decentralized applications is cryptocurrency exchange where there is no central authority which makes it easy for traceability and audit-ability.

## Pros and Cons of DApps

Pros

- Immutability - Once a transaction is made, it cannot be edited. This is possible today for all update models since data storage is relatively cheap compared to back in the day. It is possible to have an immutable audit trail of transactions, even if there are updates along the way.
- Transparency - Instead of one central authority with our data it spans across multiple network of users, which means it is virtually impossible for fraudulent transactions to slip through.
- Efficiency - Decentralized applications remove intermediaries which in turn reduces costs.

Cons

- Transactions are slow - In a production environment, blockchain transactions can take upwards of five minutes, which can get really annoying.
- High energy consumption - Each transaction made onto the blockchain requires compute through what is called ‘mining’, in order to verify and validate the transactions on each node. This is also called a consensus mechanism and contributes negatively to the environment.
- Technical complexity - Managing private keys and interacting with decentralized systems can be complex for people with a non-technical background. There is also a lot of overhead investment needed for non-decentralized apps to follow a decentralized model once they have already taken off.

## Kafka and Blockchain

After some research and playing around, I found that Kafka can solve some the problems. It gives us the speed, ability to ingest a lot of transactions at once and a way to replay all messages without adding unnecessary traffic to the blockchain. There are a lot of parallels between Kafka and the blockchain, for example the immutable commit log.

In this model, the kafka broker maintains communication with the user interface, the blockchain as well as the payment gateway. There are two active topics, one for unverified transactions and one for verified transactions.The goal is to keep them in sync.

## Code

The full code can be found in this repository - https://github.com/gucci-ninja/KafkaSummitDemo.

The `patentrail-fe` folder is the front-end, built in React using bootstrap. It can be run against a kafka-blockchain backend and a non-kafka blockchain backend. It is agnostic of the underlying backend technology as long as they vend the same API contract. It is a patent registry that can `GET /patents` and `POST /patent`.

For development purposes , I run a local Ethereum blockchain using Ganache. There is a forced delay added to mimic a production network. When running locally, is is generally instantaneous. Ganache also provides test accounts or walllets.

To deploy the application rules to the blockchain, I created a smart contract in the `smart-contracts` folder. The smart contract tells the application what we can and can’t do. The code below tells the blockchain that there is a Patent structure with an id, description and owner. Owner is an address that will map to the test account provided by Ganache. 

```solidity
pragma solidity ^0.5.0;

contract Patents {

  struct Patent {
    uint id;
    string description;
    address owner;
  }

  uint public numPatents = 0;
  mapping(uint => Patent) public patents;

  function newPatent(string memory _description) public {
    patents[numPatents] = Patent(
      numPatents,
      _description,
      msg.sender
    );
    numPatents++;
  } 
}
```

I am also using truffle library to translate the smart contract into a meaningful API. It compiles the contract to JSON, which is how the back-end will understand the rules of our application. One thing to note is that every time the contract is updated, the records on the blockchain becomes invalid. So it is very important to have a solid contract before rolling it out onto a production environment. 

First, I run the backend without kafka, in the `patentrail-be` folder. Adding a patent takes almost 10 seconds. Using a [script](https://github.com/gucci-ninja/KafkaSummitDemo/blob/main/no-kafka-script.sh) that adds five patents back to back, it takes even longer for the patent to appear. 

The next step is to switch over to the back-end implementation that leverages kafka (in folder `kafka-blockchain-be`). There are two topics, one for verified transactions and one for unverified. Transactions made by the user will immediately go into the unverified topic, whereas transactions will only be added to the verified topic once a receipt for verification is received from the blockchain. 

To ensure that transactions don’t get written to VERIFIED topic before UNVERIFIED, there is a check ensuring that VERIFIED topic is always in sync or at least behind UNVERIFIED topic.

The consumer listens to both topics. If there is a new unverified transaction, it gets added to the patents list. Once we get notified that it has been verified, we grab the patent and set verified to true. 

After running the back-end that uses kafka, there are no patents available. That is because kafka is not synced with the blockchain. So, we perform a backfill.

```solidity
module.exports.syncBlocks = async function () {
  const numPatents = await PatentContract.methods.numPatents.call().call();
  for (var i = 0; i < numPatents; i++) {
    const patent = await PatentContract.methods.patents(i).call();
    const buffer = new Buffer.from(JSON.stringify(patent));
    publish(buffer, constants.UNVERIFIED_TOPIC);
    publish(i, constants.VERIFIED_TOPIC);
  }
}
```

With this setup, we get an almost instantaneous settlement of transactions with the `Block added` status, which eventually conforms to `Verified` status.

## Final Remarks

This model has a chance of working in a trustful environment. Still a long way to go until decentralized apps become the norm but until then we need to keep experimenting.

For the full talk - please check out https://www.confluent.io/ja-jp/events/kafka-summit-apac-2021/blockchain-and-kafka-a-modern-love-story/