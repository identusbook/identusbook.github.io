#  DIDComm {#sec-didcomm}

## Overview

**DIDComm v2** (*Decentralized Identifier Communication version 2*) is a set of communication protocols designed to enable secure, private, and interoperable messaging between DIDs. These protocols have been adopted throughout Identus to standardize many important interactions, like sending and receiving encrypted messages between a mediator its peers.

[DIDComm Messaging](https://identity.foundation/didcomm-messaging/spec/) messages are encrypted JSON Web Message ([JWM](https://datatracker.ietf.org/doc/html/draft-looker-jwm-01)) envelopes. They have a standard structure including headers, a body, and optional attachments, which are encrypted and signed to ensure security and integrity.

For details on how messages are sent and received in Identus, see [Mediators](../section4/mediator.md)

## Example Project

### Swift SDK

TODO: Example code for send/receive DIDCOMM messages on Swift SDK

### TypeScript SDK

TODO: Example code for send/receive DIDCOMM messages on TypeScript SDK