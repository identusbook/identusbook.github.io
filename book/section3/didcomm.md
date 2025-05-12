#  DIDComm {#sec-didcomm}

## Overview

**DIDComm** (*Decentralized Identifier Communication*) is a crucial component in the Self-Sovereign Identity (SSI) ecosystem, providing the secure messaging layer that enables Decentralized Identifiers (DIDs) to interact. The current specification, **DIDComm v2**, evolved from earlier iterations to address the growing needs for privacy, security, and interoperability in digital identity communications. Its fundamental purpose is to enable secure, private, and authenticated communication between entities identified by DIDs, regardless of their underlying infrastructure or technology stack, transforming DIDs from static identifiers into dynamic communication endpoints capable of engaging in rich interactions.

Within the SSI technology stack, DIDComm sits above the foundational DID layer (which provides identifiers and cryptographic material) and below the application-specific protocols that define particular interactions like credential exchange. This positioning makes DIDComm the connective tissue of the SSI interactions in Identus, enabling all higher-level identity interactions.

## Key Characteristics

DIDComm's design incorporates several distinctive characteristics that make it particularly well-suited for decentralized identity communications:

**Transport agnosticism** allows DIDComm messages to travel over virtually any communication channel—HTTP(S), WebSockets, Bluetooth, NFC, mesh networks or even QR codes. This flexibility means that identity interactions aren't tied to specific network infrastructures, enabling truly peer-to-peer communications even in offline or intermittent connectivity scenarios.

**End-to-end encryption** ensures that message contents remain confidential between the sender and intended recipient(s), even when traversing untrusted networks or intermediaries. Using public key cryptography derived from DIDs, DIDComm creates secure communication channels that protect sensitive identity information.

**Authentication and trust** are inherent in DIDComm, as every message can be cryptographically verified to have originated from a specific DID. This authentication happens without centralized authorities, allowing parties to establish trust directly with one another.

**Asynchronous capabilities** enable communications that don't require both parties to be online simultaneously. Messages can be queued, stored, and forwarded until they reach their destination, making DIDComm suitable for a wide range of real-world scenarios where continuous connectivity isn't guaranteed.

**Message routing and forwarding** mechanisms allow communications to traverse complex paths, including through mediators and relays, while maintaining end-to-end security. This capability is essential for entities behind firewalls or NATs, or for preserving additional privacy by obscuring network-level metadata.

## Message Structure

[DIDComm Messaging](https://identity.foundation/didcomm-messaging/spec/) messages are structured as encrypted JSON Web Message ([JWM](https://datatracker.ietf.org/doc/html/draft-looker-jwm-01)) envelopes. This standardized format ensures interoperability while providing the necessary security properties.

A typical DIDComm message consists of:

- **Headers**: Metadata about the message, including IDs, types, timing information, and routing details
- **Body**: The actual content or payload of the message
- **Attachments**: Optional additional data, which might include structured data, documents, or media

These components are assembled into a JWM, which is then typically encrypted and/or signed according to the security requirements of the interaction. The resulting structure provides a consistent envelope format that can carry any type of content while ensuring its security and integrity.

Each message contains a unique identifier and can reference other messages through threading mechanisms, enabling complex multi-message conversations. Messages also declare their type, which connects them to specific protocols that define the expected sequence and semantics of the interaction.

## Protocols

DIDComm defines not just message formats but also a framework for protocols—standardized sequences of messages that accomplish specific tasks. These protocols enable interoperability by ensuring that different implementations understand the same message sequences and semantics.

The protocol framework includes mechanisms for discovering which protocols an agent supports and for versioning protocols as they evolve. This allows for graceful upgrades and backward compatibility in the ecosystem.

Several common protocols have emerged in the SSI ecosystem:

- **Connection establishment protocols** define how two parties can establish a secure DIDComm channel
- **Credential issuance protocols** standardize how verifiable credentials are offered and issued
- **Credential presentation protocols** define how credentials can be requested and presented
- **Trust ping protocols** provide simple mechanisms to verify that a communication channel is active

By standardizing these interaction patterns, DIDComm enables different implementations to work together seamlessly, fostering an open ecosystem rather than siloed solutions.

You can find all currently published protocols in [here](https://didcomm.org/search/?page=1), as these are maintained by the communitity, you are also encouraged to build and define your own protocols that fit your particular use case, even if you may not decide to publish them.

## Example interaction

To illustrate how DIDComm works in practice, consider a typical credential issuance scenario:

1. An Issuer and Holder establish a DIDComm connection, exchanging DIDs and service endpoints
2. The Issuer sends an offer message indicating which credentials it can provide
3. The Holder responds with a request message for a specific credential
4. The Issuer creates the credential and sends it in an issuance message
5. The Holder acknowledges receipt with an acknowledgment message

Throughout this interaction, all messages are encrypted end-to-end, ensuring that even if they pass through intermediaries, the content remains private between the Issuer and Holder. The messages might travel over different transport mechanisms—perhaps starting with an HTTPS connection but switching to Bluetooth for later messages if the parties come into proximity.

In mediated scenarios, the flow becomes more complex but even more powerful. A Holder might receive messages through a mediator service that provides consistent message delivery even when the Holder's device is offline. The mediator receives encrypted messages, stores them until the Holder connects, and then forwards them without ever being able to read the encrypted contents.

## Benefits

DIDComm delivers several crucial benefits that advance the core principles of Self-Sovereign Identity:

**Privacy preservation** stands at the forefront of DIDComm's design. By encrypting message contents and enabling pseudonymous interactions through pairwise DIDs, DIDComm allows individuals to control exactly what information they share and with whom. The transport-agnostic nature also helps prevent correlation through network metadata.

**Decentralized communication** frees identity interactions from dependence on centralized messaging providers or identity hubs. Parties can communicate directly or through mediators of their choosing, avoiding vendor lock-in and single points of failure.

**Interoperability** between different SSI implementations ensures that the ecosystem remains open and competitive. A Holder using one wallet implementation can interact seamlessly with verifiers and issuers using entirely different software stacks, as long as they all support the DIDComm protocols.

**User control** extends beyond just identity data to encompass the communication channels themselves. Individuals can choose how, when, and where they receive communications related to their digital identity, reinforcing the self-sovereign nature of the system.

## DIDComm in Identus

Identus has adopted DIDComm v2 as its standard communication protocol, implementing it throughout the framework to enable secure, private messaging between DIDs. This implementation standardizes many important interactions, including the critical communication between mediators and their peers.

The integration with mediators is particularly important in the Identus architecture. Mediators serve as message handlers that can receive, store, and forward DIDComm messages, enabling asynchronous communication even when recipients are temporarily offline. This capability is essential for practical, real-world identity solutions that must function reliably across diverse connectivity scenarios.

At the moment of writing this chapter Identus supports DIDComm over HTTP endpoints by polling, which is by far the most tested and stable implementation, but also via WebSockets enabled as a feature flag in the Cloud Agent.

For details on how messages are sent and received in Identus via mediators, see [Chapter @sec-mediator].

## Example Project

### Swift SDK

TODO: Example code for send/receive DIDCOMM messages on Swift SDK

### TypeScript SDK

TODO: Example code for send/receive DIDCOMM messages on TypeScript SDK
