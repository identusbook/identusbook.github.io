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
We will cover [mediators](/section4/mediator.html) in detail later on. For now what you need to understand is that they are used to relay messages between peers due to the nature of peers not being always online. A useful mental model for this is email, you fetch your emails from your provider when you go online so you don't require your device to be always connected in order to receive emails. Email servers as well as mediators are expected to always be online.
:::


**TODO Checklist**

- [ ] Concept of Connections
- [ ] Explain PeerDIDs
- [ ] How connections are achieved trough PeerDIDs
- [ ] Out of Band invites
- [ ] How to Connect two agents
- [ ] How to Connect edge client to agent
- [ ] Manual accept
- [ ] Auto accept
- [ ] DIDless connection (Atala Roadmap)
