---
title: "Connections"
---
## Overview

Now that we have a better understanding of Wallets and DIDs, it's time to embark on our first interaction. In this chapter we are going to explore conceptually what a Connection means in SSI, take a deep dive into DID Peers, explain how they work and why they are needed for secure connections, dissect Out of Band invites and finally hands on example code to achieve connecting edge client to an agent.

Before we move forward we highly recommend to at least read the basic [Connection tutorial](https://docs.atalaprism.io/tutorials/connections/connection) on the official [Atala documentation](https://docs.atalaprism.io/).

## Connections in Self-Sovereign Identity

Connections are fundamental to establishing *trusted interactions* between peers. They enable secure and verifiable communication, allowing entities to exchange credentials and proofs in a decentralized manner. This relationship is established using a specific decentralized identifier standard (**Peer DID**) and is governed by a protocol (**DIDCOMM**) that ensure the authenticity, integrity, and privacy of the interactions between the connected parties.

There are three roles in an SSI connection:

- **Inviter**: The entity that initiates the connection by sending an invitation.
- **Invitee**: The entity that receives the invitation and responds with a connection request.
- **Mediator**: An intermediary that facilitates message delivery between entities, especially when one or both parties may not always be online.

::: {.callout-note}
We will cover [mediators](/section4/mediator.html) in detail later on. For now what you need to understand is that they are used as a service to relay messages between peers, they will store messages and deliver them whenever a peer comes back online, connects to the mediator and fetches their messages. 
:::

## PeerDIDs

They are a special kind of decentralized identifier with some unique properties that allow them to be perfect for use in order to establish private and secure communications between peers.

DID Documents such as PrismDIDs are meant to be publicly available and resolvable by arbitrary parties, therefor storing them in a VDR such as Cardano blockchain is an excellent way to achieve this requirement in a reliable way. 

However, when Alice and Bob want to interact with each other, only two parties care about the details of that connection: Alice and Bob. Instead of arbitrary parties needing to resolve their DIDs, only Alice and Bob do. Thus, PeerDIDs essentially describe a key-pair to encrypt and sign data to and from Alice and Bob, routed trough their preferred mediators, e.g. When Alice accepts an invite from Bob and they engage the connection protocol, Alice generates a PeerDID that allows her to encrypt and sign data routed trough Bob's mediator (mediator Y) that only Bob can decrypt, and viceversa, Bob will generate a PeerDID that allows him to encrypt and sign data routed trough Alice's preferred mediator (mediator X) that only Alice can decrypt.

The key benefits of PeerDIDs are:

1. Decentralized by nature.
2. No transaction cost on blockchain.
3. Private (only the concerned parties know about them).
4. Reusable without any reliance on the internet, with no degradation of trust. (adheres to the principles of [local-first](https://www.inkandswitch.com/local-first/) and [offline-first](https://offlinefirst.org))

To go deeper in your understanding of PeerDIDs please refer to the full [Peer DID Method Specification](https://identity.foundation/peer-did-method-spec). In the Hyperledger Identus ecosystem, only PeerDIDs method 2 are supported at the time of this writing.

## Out of Band invites

**TODO Checklist**

- [x] Concept of Connections
- [x] Explain PeerDIDs
- [x] How connections are achieved trough PeerDIDs
- [ ] Out of Band invites
- [ ] How to Connect two agents
- [ ] How to Connect edge client to agent
- [ ] Manual accept
- [ ] Auto accept
- [ ] DIDless connection (Atala Roadmap)
