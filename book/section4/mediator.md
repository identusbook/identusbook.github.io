# Mediator {#sec-mediators}

## Overview

A Self-Sovereign Identity Mediator is a service that enables private and secure connections between peers. Its primary function is to relay encrypted messages between DIDs using [DIDComm V2](../section3/didcomm.md) protocols. Since each participant wallet is Self-Sovereign, Mediators help route messages to their intended recipient without any one wallet knowing the details of the other.  Encrypted messages can be sent to peers even if they are not online, and the Mediator will manage a queue for the offline party until they are able to connect and retrieve them.

Think of a Mediator like a set of Post Office mailboxes. Each mailbox has a mailing address (*a DID*).  Each mailbox can hold an stack of incoming mail [Encrypted DIDComm V2 messages](../section3/didcomm.md) for its owner.  Each mailbox also keeps a list of cryptographic [Connections](../section3/connections.md) with whom its owner can send or receive mail with.  In this way, Identus Mediators facilitate all types of important transactions, including routing both messages, Verifiable Credential issuance, and Credential Exchange.

What's important is that the Mediator, while messages are routed through it, is not a centralized actor.  Mediators can not know the contents of any message, only what DIDs can talk to other DIDs and what encrypted messages they have yet to read.

### Why use a Mediator?

Why do we need a Mediator in the first place?  If participants are Self-Sovereign, then why not connect an edge wallet directly to another edge wallet?  

SSI participants are connected through various types of devices and network conditions.  [The Cloud Agent](../section2/installation-local.md), is expected to be online and available all the time, as it's hosted in a remote server farm somewhere.  However users who keep their wallets on mobile devices, or laptop computers, are only online sporadically, and can't be expected to have a reliable connection to the Internet at all times.  This is where Mediators earn their keep.  If User A were to send a message to User B, who happens to be offline at the moment, the message could not be delivered, plus User A would also have to know sensitive data about how to contact User B. If User B changes devices, they would have to tell User A, and all other peer Connections.  If User A instead, sends the message through the Mediator, it can be held securely and retrieved the next time User B connects to the Internet. Mediators provide reliability between [Connections](../section3/connections.md), without compromising privacy or security.  

## Set up a Mediator

We will teach the reader how to

- Install, configure, and Run your own Mediator
- How to manage Websockets
- Performance Tuning
