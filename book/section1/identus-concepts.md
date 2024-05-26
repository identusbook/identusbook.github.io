---
title: "Identus Concepts"
---

Identus Application Architecture Diagram

- Cloud Agent
- Edge Agent
- PRISM Node
- Building Blocks
- Mediator
- DIDComm2


Identus is made up of several open source components.  Each could be used or forked separately but they are designed to work well together.

**PRISM Node:**

PRISM Node implements the `did:prism` methods and is an interface to multiple [VDR (Verifiable Data Registries)](../glossary.md#vdr).  The node can resolve PRISM DIDs and write transactions to a blockchain or database. PRISM Node is expected to be online at all times.

**Cloud Agent:**

Written in Scala, the Cloud Agent runs on a server and communicates with clients and peers via a REST API.  It is a critical component of an Identus application, able to manage identity wallets and their associated operations, as well as issue Verifiable Credentials.  The Cloud Agent is expected to be online at all times.

**Edge Agent:**

Edge Agents give agent capabilities to clients like Websites and mobile apps.  They can can never be assumed to be online at any given time, and therefore rely on sending and receiving all communications through an online proxy, the Mediator.

**DIDComm:**

DIDComm is a private, secure, and interoperable protocol for communication between decentralized identities. 
Identus supports DIDCommV2 and allows peers to pass messages between each other, proxied by the Mediator. Messages contain a `to:` and `from:` DID

**Mediator:**

Mediators act as middlemen between Peer DIDs.  In order for any agent to send a message to any other agent, it must know the `to` and `from` DIDs of each message. The sender and recipient together make up a cryptographic connection called a `DIDPair`.  Mediators maintain queues of messages for each `DIDPair`. If an Edge Agent is offline, the Mediator will hold incoming messages for them until the agent is back online and able to receive them. Mediators can deliver messages when polled, or push via Websockets. Mediators are expected to be online at all times and be highly available.

Since instantiation of Identus Edge Agents requires a Mediator, there are several publicly available Mediator services which make development simple.  

- [PRISM Mediator](https://github.com/input-output-hk/atala-prism-mediator)
- [RootsID Mediator](https://github.com/roots-id/didcomm-mediator)
- [Blocktrust Mediator](https://github.com/bsandmann/blocktrust.Mediator)

While extremely helpful during development, these are not recommended for production Identus deployments as they have no uptime guarantee and will not scale past a small number of concurrent users.  We will discuss how to run your own Mediator in [Chapter @sec-mediators]

**Building Blocks:**

Known as the "Building Blocks", there are several open source libraries that are responsible for the underlying technology Identus applications depend on, like cryptography operations and 