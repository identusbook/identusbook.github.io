---
title: "Wallets"
---
## Introduction

Wallets are an essential component on every Self-Sovereign Identity interaction, and as you might have guessed, just like in the physical world a wallet holds your identifiers (IDs), in the digital SSI realm their function is to store and manage Decentralized Identifiers (DIDs), Verifiable Credentials (VCs), cryptographic keys, and other related assets.

Since many SSI frameworks rely on a Blockchain to publish DIDs, there is a common misconception that SSI wallets work in a similar way. Although both SSI and Blockchain wallets require a `seed phrase` built from a random set of `mnemonics`, thats where the overlap ends, because the balance and history of a Blockchain wallet can be restored from the ledger itself as opposed to an SSI wallet, where all the stored information exists only on the device and needs to be manually backed up and restored.

In essence, a wallet in SSI is piece of software that allows users to store, manage, and present proof of their digital identities and credentials. It acts as a repository for digital assets required to fulfill every SSI interactions, ensuring that users and entities have complete control over their data. In the case of Identus, the [Edge Agent SDK](https://github.com/hyperledger/identus-edge-agent-sdk-ts) provides all the abstractions needed to operate a wallet by an individual on a browser or mobile app, and the [Cloud Agent](https://github.com/hyperledger/identus-cloud-agent), provides a `REST API` to operate a wallet in the cloud, either to itself (single tenant) or for third-parties (multi tenant) as a custodial wallet.

## How wallet works in Identus


TODO:
- [x] Explain the difference between a blockchain wallet and an identity wallet
- [ ] How wallet works in Identus
- [ ] Custodial wallets
- [ ] Pluto Encrypted
- [ ] Keycloak examples
- [ ] Recovery