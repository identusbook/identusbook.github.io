---
title: "Wallets"
---
## Overview

Wallets are an essential component on every Self-Sovereign Identity interaction, and as you might have guessed, just like in the physical world where a wallet holds your identifiers (IDs), a digital SSI wallet's function is to store and manage Decentralized Identifiers (DIDs), Verifiable Credentials (VCs), cryptographic keys, and other related assets.

Since many SSI frameworks rely on a Blockchain to publish DIDs, there is a common misconception that SSI wallets work in a similar way. Although both SSI and Blockchain wallets require a `seed phrase` built from a random set of `mnemonics`, thats where the overlap ends, because the balance and history of a Blockchain wallet can be restored from the ledger itself as opposed to an SSI wallet, where all the stored information exists only on the device and needs to be manually backed up and restored.

In essence, a wallet in SSI is piece of software that allows users to store, manage, and present proof of their digital identities and credentials. It acts as a repository for digital assets required to fulfill every SSI interaction, ensuring that users and entities have complete control over their data. In the case of Identus, the [Edge Agent SDK](https://github.com/hyperledger/identus-edge-agent-sdk-ts) provides all the abstractions needed to operate a wallet by an individual on a browser or mobile app, and the [Cloud Agent](https://github.com/hyperledger/identus-cloud-agent), provides a `REST API` to operate a wallet in the cloud, either to itself (single tenant) or for third-parties (multi tenant) as a custodial wallet.

## Edge Agent SDK in Identus

Identus provides it's wallet interface through the Edge Agent SDKs, available in 3 flavors:

- [TypeScript](https://github.com/hyperledger/identus-edge-agent-sdk-ts) for Web and Node apps.
- [Swift](https://github.com/hyperledger/identus-edge-agent-sdk-swift) for iOS and Mac.
- [Kotlin](https://github.com/hyperledger/identus-edge-agent-sdk-kmp) for Android and JVM.

Each of the flavors provide the same building block implementations:

- Apollo: Provides a suite of necessary cryptographic operations.
- Castor: Provides a suite of operations to create, manage and resolve decentralized identifiers.
- Pollux: Provides a suite of operations for handling [verifiable credentials](https://github.com/hyperledger/identus-docs/blob/master/documentation/docs/concepts/glossary.md#verifiable-credentials).
- Mercury: Provides a suite of operations for handling DIDComm V2 messages.
- Pluto: Provides an interface for storage operations in a portable, storage-agnostic manner.
- Agent: A component using all other building blocks, provides basic edge agent capabilities, including implementing DIDComm V2 protocols.

And all of them abstract the usage of each building block through the Agent component.

Most of the time you will be operating the wallet trough the Agent interface unless you require to directly call lower level building block API, for example, you may require to send a custom DIDCOMM message payload format which is not directly supported via the Agent building block, you can still use Mercury directly to achieve that. This approach gives you a simple to use interface trough the Agent but also the flexibility and control that comes with also providing access to the lower level APIs. We highly encourage to dig inside each of the building blocks and study how the Agent is using them, this will come handy when you need to add custom features to your own software.

There is one building block that is *not* implemented and only provided as an interface, this is Pluto, the storage layer for DIDs, VCs, messages, keys, etc. Identus does not have an opinion on how you should store and retrieve the contents of the wallet, so it's your job to implement this part according to your needs. Fortunately, there is a community project providing one implementation called [Pluto Encrypted](https://github.com/atala-community-projects/pluto-encrypted), this project provides 3 different storage engines: InMemory, IndexDB, LevelDB. As the name suggest, Pluto Encrypted provides full Pluto compatibility plus handles encryption and decryption of the wallet contents, this is very important due to the fact that the wallet stores your DIDs (private keys), VCs, messages and a lot of sensitive information. If you are starting out we highly recommend you to use this implementation before attempting to role your own, it's a great starting place that you can extend and customize to your needs.

## Custodial Wallets

In an ideal world, everyone should be willing and able to manage their own identity wallets, this is one of the main characteristics of truly Self-Sovereign ecosystem. In practice, there are many good reasons why an identity wallet would be better managed by a service. Such is the case for companies and entities or even individuals that don't want to deal with the responsibility and risk of self-managing their wallets. For this use case Identus provides the concept of Custodial Wallets. What this really means is that an identity wallet can be managed by the Cloud Agent and used over a REST API. For this particular use case, the Cloud Agent supports a multi-tenant mode in order to onboard and serve multiple identity wallets on the same running instance. We will explain the setup in detail over the production installation section, for now the key insight is that when you access your identity wallet through a Cloud Agent, you are really trusting the storage and management of the private keys of your identity to that service.