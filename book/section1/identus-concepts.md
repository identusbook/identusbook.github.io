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

PRISM Node:
    PRISM Node implements the `did:prism` methods and is an interface to multiple VDR (Verifiable Data Registries).  The node can resolve PRISM DIDs and write transactions to a blockchain or database. [More to write here] PRISM Node is expected to be online at all times.

Cloud Agent:
    Written in Scala, the Cloud Agent runs on a server and communicates over a REST API.  It is a critical component of an Identus application, able to manage identity wallets and their associated operations, as well as issue Verifiable Credentials.  The Cloud Agent is expected to be online at all times.

Edge Agent:
    Edge Agents live on the client side, closer to the end user.  Because this can traditionally be assumed to be a mobile or web client application, it can never be assumed to be online at any given time.  Edge Agents communicate using the Mediator as a proxy

Mediator:
    Mediators act as middlemen between Peer DIDs.  In order for any agent to send a message to any other agent, it must know the To and From DIDs of each message. The sender and receipient together make up a cryptograhpic connection called a DIDPair.  Mediators maintain queues of messages for each DIDPair. If an Edge Agent is offline, the Mediator will hold incomming messages for them until the agent is back online and able to receive them.      Mediators are expected to be online at all times and be highly available.

DIDComm:

Building Blocks:
    Known as the "Building Blocks", there are several open source libraries that are responsible for the underlying technology Identus applications depend on, like cryptography operations and 