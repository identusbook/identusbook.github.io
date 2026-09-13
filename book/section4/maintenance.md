# Maintenance {#sec-maintenance}

Mastering Identus means maintaining your application once it is launched. The production deployment from [Chapter @sec-installation-production] is not a single process; it is a set of long-running services — Cloud Agent, PRISM Node (or NeoPRISM), HashiCorp Vault, PostgreSQL, and, in Cardano mode, a Cardano node, Cardano Wallet, and Cardano DB Sync — that each have their own lifecycle, storage, and failure modes. This chapter covers the day-two work of keeping that stack healthy: restarting it cleanly, observing it, upgrading it without breaking compatibility, and protecting the secrets and keys it depends on.

This chapter assumes the deployment described in the production installation chapter. Where a topic is well-grounded in that deployment, the guidance below is concrete. Where Identus operational practice is still immature or undocumented, the section is marked as a placeholder to expand in a later draft.

## Restarting and Cleanup

The services in a Cardano-mode deployment have a dependency order, and restarts should respect it. Cardano Wallet and Cardano DB Sync both connect to the Cardano node through its node socket, PRISM Node depends on Cardano Wallet and DB Sync, and the Cloud Agent depends on PRISM Node and PostgreSQL. A clean restart generally moves outward from the ledger toward the application:

1. Start (or confirm) PostgreSQL and unseal Vault. The Cloud Agent cannot read wallet seeds while Vault is sealed.
2. Start `cardano-node`, then `cardano-wallet` and `cardano-db-sync` once the node socket is available.
3. Wait for DB Sync to reach close to chain tip before relying on `did:prism` publication or resolution.
4. Start PRISM Node, then the Cloud Agent, then the controller application.

A restart does not re-anchor existing DIDs. The Cloud Agent reconstructs DID key material from the wallet seed and stored derivation path, so seed availability (Vault unsealed and reachable) is the precondition for issuance and DID-update flows after a restart.

Cleanup is mostly about disk and logs. The Cardano services produce continuous logs during synchronization; keep log rotation configured for every long-running container, not just the Cardano ones. DB Sync's PostgreSQL database grows continuously and is the largest consumer of disk in the stack. Unlike the Cloud Agent and PRISM Node databases, DB Sync data is derived chain state: it can be rebuilt by resyncing or restored from a trusted snapshot, so it is a candidate for pruning or rebuild rather than long-term backup.

> TODO: Add concrete cleanup runbooks — pruning Docker images and volumes, reclaiming DB Sync disk, and the resync-vs-snapshot decision with timings on `preprod` and `mainnet`.

## Observability

Operating Identus well means watching both the application layer (Cloud Agent and controller) and the ledger layer (Cardano services), because a problem in either can stall credential issuance while leaving the Cloud Agent itself "up."

### Health and Service Checks

The production check list in [Chapter @sec-installation-production] is the starting point for ongoing monitoring. The checks worth running continuously, not just at launch, include:

- Cloud Agent health returns the expected version, and the public `REST_SERVICE_URL` answers over HTTPS.
- `DIDCOMM_SERVICE_URL` is reachable from holder wallets and mediator services.
- Cardano DB Sync is close to chain tip. A DB Sync that falls behind silently delays DID publication and resolution.
- Cardano Wallet reports a healthy network state (`/v2/network/information`) and, on mainnet, the PRISM Node payment address holds enough ADA for transaction fees.
- An end-to-end probe: a `did:prism` create operation reaches confirmed state and then resolves in short form.

### Managing Nodes and Memory

Cardano DB Sync is the heaviest resource consumer in the stack, and its requirements dominate sizing. The production chapter records mainnet figures on the order of 64 GB RAM, 4+ CPU cores, SSD storage at 60k IOPS or better, and several hundred GB of disk that grows over time; `preprod` is far smaller. Run DB Sync and its PostgreSQL database close together (ideally colocated) to keep synchronization latency low, and monitor database IOPS if services are split across machines.

For ongoing operation, watch:

- DB Sync PostgreSQL disk growth and remaining headroom.
- `cardano-node` and `cardano-db-sync` memory against the host's limits.
- Cloud Agent and PRISM Node database size and query latency, which affect issuance and verification throughput.

### Performance Testing

> TODO: Placeholder. The book does not yet define a performance-testing methodology for Identus. Expand this section with a repeatable load test (issuance, verification, and DID-resolution throughput), the metrics to capture, target numbers, and how Cardano confirmation depth bounds end-to-end issuance latency.

### Analytics with BlockTrust Analytics

> TODO: Placeholder. BlockTrust Analytics is a third-party analytics tool for Identus/PRISM. Expand this section with what it observes, how to connect it to a running deployment, and which operational questions it answers that the raw health checks do not.

## Upgrading Agents

### Version Selection and Compatibility

Treat the Identus platform release notes as the first compatibility source. A platform release pins a set of components that are known to work together — for example, a Cloud Agent version, a Mediator version, a NeoPRISM version, and an SDK version. Newer standalone component releases (a newer Cloud Agent or PRISM Node image) may exist, but they require compatibility testing against the platform pin: untested Cloud Agent and PRISM Node image pairs can fail compatibility checks.

In practice, upgrades change one or more of these pinned versions:

- `AGENT_VERSION` (Cloud Agent)
- `PRISM_NODE_VERSION` (PRISM Node) or the NeoPRISM backend version
- `CARDANO_WALLET_TAG` and its paired `cardano-node` version
- `CARDANO_DB_SYNC_VERSION`

Whenever you change any Identus image, rerun the full production check list before depending on the deployment.

### Upgrade Procedure and Compatibility Notes

Cardano DB Sync upgrades deserve special care: read the release notes before changing the image tag. DB Sync upgrades can run schema migrations (for example, an `epoch` table migration between point releases), and snapshot compatibility is release-specific. Validate a DB Sync upgrade on `preprod` before applying it to `mainnet`, and confirm the new version still reaches chain tip and that PRISM Node can publish and resolve a test DID afterward.

For Cloud Agent and PRISM Node, confirm the pair is a tested combination, then upgrade and run an end-to-end issuer flow (create issuer DID, issue, optionally revoke/suspend, present, verify) before cutting traffic over.

### Minimizing Downtime

> TODO: Placeholder. Document a low-downtime upgrade strategy. Open questions to resolve before writing this with confidence: whether the Cloud Agent supports running old and new versions side by side against a shared database, how database migrations are applied across an upgrade, and how to drain in-flight DIDComm and issuance flows. Until then, plan for a maintenance window.

## HashiCorp Vault and Key Management

### Key Management

For production, the Cloud Agent stores wallet seed material in Vault (`SECRET_STORAGE_BACKEND=vault`), under wallet-scoped paths such as `/secret/<wallet-id>/seed`, along with peer DID key paths and other secrets. A tenant's wallet seed is what lets the agent re-derive DID keys after a restart or redeploy; losing it can make existing DIDs unusable for future updates, deactivation, or issuance. Vault is therefore the single most important thing to protect and back up in the deployment.

Operational essentials carried over from the production chapter:

- Use AppRole (`VAULT_APPROLE_ROLE_ID` / `VAULT_APPROLE_SECRET_ID`) for automated deployments. Reserve `VAULT_TOKEN` for lab or break-glass use.
- Run Vault with production hardening: TLS, an unprivileged runtime user, a deliberate memory-lock/encrypted-swap decision, storage separation, and restricted network paths. A `server -dev` Vault is lab storage only.
- Back up Vault storage, test the restore, and document the unseal or recovery-key procedure. After any Vault restart, the store must be unsealed before the Cloud Agent can read seeds.

### Backups and Restore

Back up by data type, because the services do not all need the same treatment:

- **Cloud Agent and PRISM Node databases** hold application state. Back them up and test restores.
- **Vault storage** holds wallet seeds and key material. Back it up, test restore, and protect the unseal/recovery keys.
- **Cardano Wallet mnemonic and spending passphrase**, and the PRISM Node payment address, belong in the production secret system — never in `.env` files, CI logs, shell history, or the repository.
- **Cardano DB Sync database** is derived chain state. It can be rebuilt by resyncing or restored from a trusted snapshot, so it is lower priority for backup than application state.

### Key and Credential Rotation

Rotate operational credentials through the same process used for the rest of your production secrets: `ADMIN_TOKEN`, tenant API keys, database passwords, Vault credentials, Keycloak client secrets, and the Cardano Wallet passphrase.

DID key rotation is a different, protocol-level operation. A `did:prism` controller rotates verification keys by publishing a signed DID **update** operation through PRISM Node, which changes the DID Document's verification methods and relationships on-chain (see [Chapter @sec-did-and-diddocuments]). Because issuance and DID updates depend on key material derived from the wallet seed, key rotation and seed custody are tightly linked.

> TODO: Placeholder for a concrete rotation runbook. Expand with step-by-step procedures and their blast radius: rotating the wallet seed and what happens to existing DIDs, rotating `did:prism` verification keys via update operations without invalidating already-issued credentials, and rotating Vault AppRole credentials with zero downtime.
