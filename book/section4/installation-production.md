---
title: "Installation - Production Environment"
---
## Overview

A production environment setup requires connecting Hyperledger Identus to the Cardano blockchain as the Verifiable Data Registry (VDR). This is achieved through the `prism-node` component, which abstracts the VDR operations for publishing, resolving, updating, and deactivating Decentralized Identifiers (DIDs).

According to the official documentation:

>The PRISM Node generates a transaction with information about the DID operation and verifies and validates the DID operation before publishing it to the blockchain. Once the transaction gets confirmed on the blockchain, the PRISM Node updates its internal state to reflect the changes.

While our local setup instructs `prism-node` to use a local database as the VDR for testing and development, it lacks the benefits of Cardano's secure and decentralized blockchain for publishing DIDs on-chain. To reach the blockchain, our `prism-node` needs to connect to two components:

1. A full node Cardano wallet to submit transactions to the blockchain
2. Cardano-DB-Sync to read the blockchain through a normalized database interface

When the cloud agent needs to publish, update, or deactivate DIDs, it requests `prism-node` to create and validate a transaction, which is then passed to the `cardano-wallet` for submission to the blockchain. Simultaneously, `prism-node` connects to `cardano-db-sync` to read new blocks, filter DID Prisms published on-chain, and notify the cloud agent when the DID operation reaches a certain number of confirmations.

Production and pre-production installations are similar, with the main difference being that production points to `mainnet` and pre-production to `testnet` for both `cardano-wallet` and `cardano-db-sync`. This is achieved by changing environmental variables passed to the Docker containers.

For a production setup, we recommend additional security measures, including changing default passwords, deactivating unnecessary services, and setting up managed database and secret storage providers with regular backups. While these aspects are beyond the scope of this book, we will provide corresponding notes with our recommendations when appropriate.

## Hardware recommendations

The Hyperledger Identus cloud agent alone doesn't require too much hardware, any instance with 2GB-4GB ram will run it for testing purposes. Of course as you scale in usage you will naturally want more ram available to handle higher concurrent loads.

The Cardano wallet and DB sync components on the other hand are going to need a lot more resources due to the `mainnet` requirements, according to the official documentation, to run a full Cardano node you need:

- 200GB of disk space (for the history of blocks)
- 24GB of RAM (for the current UTxO set)

Of course, on a production environment you may want to run your agent, wallet and db-sync on different machines connected through a VPN or SSH tunnel in order to isolate them and improve security, e.g., your Cardano wallet may be only connecting to a Cardano node, but not exposed to the Internet. You may also reuse and share your Cardano node instance for your wallet and db-sync components, reducing your hardware requirements.

There are many ways to setup your infrastructure and at least in the beginning, the Cardano node is the most resource demanding of all components.


---
In this chapter we will discuss how to prepare a production envioronment for an Identus application.

We will discuss:

- Hardware recommendations
- Production configuration and security
- Lock down Docker / Postgres (default password hole, etc)
- SSL
- Multi Tenancy
- Keycloak
- Testnet / Preprod env
- Connecting to Mainnet
- Set up Cardano Wallet
- Key Management with HashiCorp
- Connecting to Cardano
- Running dbSync

