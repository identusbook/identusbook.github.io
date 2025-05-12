#  Identus Concepts {#sec-identus-concepts}

[Identus Application Architecture Diagram]

Identus is made up of several open source components.  Each could be used or forked separately but they are designed to work well together.

## PRISM Node

PRISM Node implements the `did:prism` method and serves as a second-layer node for the Distributed Ledger, currently it only supports Cardano blockchain or local database but in the future it will be acting as a comprehensive interface to multiple [VDR (Verifiable Data Registries)](../glossary.md#vdr). The node can resolve PRISM DIDs and write transactions to a blockchain or database, maintaining an indexed internal state that's synchronized with the underlying blockchain for efficient lookup operations.

As a critical component in the Identus ecosystem, PRISM Node provides a secure and trustworthy platform for storing and managing decentralized identifiers. It handles the creation, update, resolution, and deactivation of PRISM DIDs by generating transactions with the necessary operation information, verifying and validating these operations, and publishing them to the blockchain. Once transactions are confirmed, the node updates its internal state accordingly.

The PRISM Node's architecture enables users to:
- Create DIDs without initially interacting with any PRISM Node, with the option to announce them publicly later. This means you can create unpublished DIDs via Apollo building block and make use of it, not all DIDs are required to be published on chain, but if you need to anchor a DID on chain, PRISM Node will handle that operation for you.
- Update DID documents by publishing update operations on chain.
- Deactivate DIDs by publishing deactivation operations on chain.
- Resolving DIDs by querying historical changes on chain.
- Track the status of operations submitted to the node.

This second-layer approach is essential for making DIDs scalable and efficient, as it provides the necessary off-chain processing and data storage capabilities while leveraging the security and immutability of the underlying blockchain. PRISM Node is expected to be online at all times to ensure reliable service.

## Cloud Agent

Written in Scala, the Cloud Agent runs on a server and communicates with clients and peers via a REST API. It is a critical component of an Identus application, able to manage identity wallets and their associated operations, as well as issue Verifiable Credentials. The Cloud Agent is expected to be online at all times.

The Cloud Agent is designed to be scalable, robust, and standards-compliant, providing comprehensive self-sovereign identity services. It supports W3C standards, DIDCommV2, and Hyperledger Aries protocols, ensuring interoperability within the broader SSI ecosystem. Key capabilities include:

- Support for multiple agent roles including Issuer, Holder, Verifier
- Management of W3C Standard Verifiable Credentials (JSON and JSON-LD formats encoded as JWT), SD-JWT and AnonCreds
- Implementation of DIF Presentation Exchange for credential requests and submissions
- Support for `did:prism` and `did:peer` DID methods
- Full implementation of DIDCommV2 messaging and protocols
- Compatibility with Aries RFCs including DID exchange, out-of-band protocol, issue credential, and present proof

The Cloud Agent's REST API enables developers to build controllers in any programming language without needing deep expertise in the underlying SSI standards. This architecture allows business logic to be separated from the identity infrastructure, making it easier to develop specialized applications while leveraging the full power of decentralized identity.

When deployed, the Cloud Agent interacts with the PRISM Node over gRPC protocol, using it as the Verifiable Data Registry to anchor DIDs on a distributed ledger for high security and availability.

## Cloud Agent Building Blocks

Identus separates the handling of important SSI operations into separate, focused libraries called "building blocks." These modular components can be combined and configured to meet various use cases and product requirements. This modular architecture provides excellent flexibility and customization options, allowing developers to implement decentralized identity solutions tailored to their specific needs.

### Apollo - Cryptography Module

Apollo is a comprehensive cryptographic primitives toolbox that Identus uses to ensure data integrity, authenticity, and confidentiality. It provides the foundation for secure communication and data protection throughout the Identus ecosystem.

Apollo employs cryptographic hash functions to create digital fingerprints of data, allowing the detection of any unauthorized modifications. For example, when a verifier receives a credential, Apollo's cryptographic functions can verify that the credential hasn't been tampered with since it was issued.

Additionally, Apollo implements digital signatures to authenticate the identity of senders and recipients, and uses encryption algorithms to protect sensitive data from unauthorized access. In a healthcare scenario, Apollo's encryption would ensure that patient credentials remain confidential when shared between authorized parties.

### Castor - DID Module

Castor enables the creation, management, and resolution of Decentralized Identifiers (DIDs). It currently supports the native `did:prism` method and the `did:peer` method but the are discussions aligning the team to build a more flexible architecture that would allow anyone to write plugins that could support other DID methods.

When a user creates a new digital identity in an Identus application, Castor generates the DID and associated cryptographic material. For instance, a university issuing student credentials would use Castor to create and manage DIDs for both the institution and potentially for each student, establishing the foundation for trusted digital relationships.

Castor's resolver component can look up a DID and retrieve its associated DID Document, which contains the public keys, authentication mechanisms, and service endpoints needed for secure interactions with that identity.

### Pollux - Verifiable Credential Module

Pollux handles all Verifiable Credential operations, allowing users to issue, manage, and verify credentials in a privacy-preserving manner. This building block is essential for implementing the core functionality of credential exchange in self-sovereign identity systems.

In a real-world employment scenario, a company could use Pollux to issue employee credentials, which employees could then store in their digital wallets. When applying for a loan, an employee could selectively disclose relevant employment information from this credential without revealing unnecessary personal details. The bank could then use Pollux's verification capabilities to confirm the credential's authenticity and validity without contacting the employer directly.

Pollux also manages credential status, enabling issuers to revoke credentials when necessary and allowing verifiers to check if a credential is still valid before accepting it.

### Mercury - DIDComm Module

Mercury provides an interface to the DIDCommV2 protocol, enabling secure, private communication between DIDs regardless of the underlying transport mechanisms. This building block establishes the foundation for all agent-to-agent communications in the Identus ecosystem.

For example, when a citizen wants to share a government-issued credential with a service provider, Mercury facilitates the encrypted, authenticated message exchange between the citizen's wallet and the service provider's verification system. This communication happens peer-to-peer, without requiring centralized intermediaries to facilitate the exchange.

Mercury's transport-agnostic design means that these secure communications can occur over various channels, including HTTP, WebSockets, or Bluetooth, making it versatile for different deployment scenarios from web applications to mobile devices.

More detailed information on each of the Building Blocks can be found in the [Identus Documentation](https://hyperledger-identus.github.io/docs/home/identus/cloud-agent/building-blocks).

## Edge Agent

Edge Agents bring agent capabilities to client applications such as websites, mobile apps, and other user-facing software. Unlike Cloud Agents, Edge Agents cannot be assumed to be online at all times, and therefore rely on sending and receiving all communications through an online proxy called a Mediator.

Edge Agents are implemented as SDKs that developers can integrate into their applications, providing a comprehensive suite of SSI functionality directly on the client side. These SDKs handle critical operations including:

- Creating and managing DIDs
- Storing and managing Verifiable Credentials
- Secure communication via DIDComm
- Cryptographic operations for signing and verification
- Local secure storage of identity data

Identus provides Edge Agent SDKs in multiple programming languages to support various platforms:

- **TypeScript SDK**: For web applications
- **Swift SDK**: For iOS applications
- **Kotlin Multiplatform SDK**: For Android applications and cross-platform development

Each SDK implements the same core building blocks (Apollo, Castor, Mercury, Pollux, and Pluto interfaces) as the Cloud Agent, ensuring consistent functionality and interoperability across the entire Identus ecosystem. This architectural consistency allows developers to create seamless experiences where identity operations can be performed either on the client or server side as appropriate for their use case.

Edge Agents typically function as digital wallets, enabling users to maintain control over their credentials and identity data on their personal devices. This approach aligns with the core principles of Self-Sovereign Identity by keeping the user in control of their data and minimizing dependency on centralized services.

## Mediator

Mediators act as middlemen between Peer DIDs.  In order for any agent to send a message to any other agent, it must know the `to` and `from` DIDs of each message. The sender and recipient together make up a cryptographic connection called a `DIDPair`.  Mediators maintain queues of messages for each `DIDPair`. If an Edge Agent is offline, the Mediator will hold incoming messages for them until the agent is back online and able to receive them. Mediators can deliver messages when polled, or push via web sockets. Mediators are expected to be online at all times and be highly available.

Since instantiation of Identus Edge Agents requires a Mediator, there are several publicly available Mediator services which make development simple.

- [PRISM Mediator](https://github.com/input-output-hk/atala-prism-mediator)
- [RootsID Mediator](https://github.com/roots-id/didcomm-mediator)
- [Blocktrust Mediator](https://github.com/bsandmann/blocktrust.Mediator)

While extremely helpful during development, these are not recommended for production Identus deployments as they have no uptime guarantee and will not scale past a small number of concurrent users.  We will discuss how to run your own Mediator in [Chapter @sec-mediator]

## Verifiable Data Registry (VDR)

The Verifiable Data Registry (VDR) system addresses the critical challenge of ensuring authenticity and integrity for publicly available data in an environment where data is generated and consumed at unprecedented scales. Traditional systems often rely on centralized authorities, introducing single points of failure and undermining trust, especially when multiple independent parties need to rely on the same data.

Key challenges include enabling decentralized verification, guaranteeing data integrity and authenticity without central oversight, ensuring interoperability across diverse storage technologies (databases, blockchains, in-memory caches), and facilitating scalable, trustless collaboration.

The VDR system tackles these issues by providing a unified API and a modular, extensible framework for storing, mutating, retrieving, and removing data. It decouples the application layer from the underlying storage mechanisms through two main pluggable components:

- **Drivers**: These are plugins that implement the actual data storage and retrieval logic. Each driver is tailored to a specific storage backend (e.g., a database, a blockchain, or in-memory storage). Drivers are responsible for executing operations like creating, reading, updating, and deleting data. They also handle the generation and verification of cryptographic proofs (hashes for immutable data, digital signatures for mutable data) to ensure data integrity.
- **URL Managers**: These components construct and resolve URLs that reference the stored data. They embed necessary metadata (like driver identifiers, driver families, and cryptographic proofs) into these URLs using standard query parameters. This allows the VDR system to select the correct driver and verify data integrity when a URL is presented.

By abstracting these operations, the VDR system ensures consistent and verifiable data operations regardless of where or how the data is stored. It employs cryptographic hashes for immutable data and digital signatures for mutable data, embedded in URLs or retrievable via the driver—to guarantee integrity. This architecture fosters a trustless environment where multiple parties can independently verify data authenticity and integrity, enhancing security and collaboration. The VDR system's design allows for adaptability to various storage backends while maintaining consistent methods for data verification and access.
