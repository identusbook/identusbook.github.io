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

Rotation covers two different kinds of secret. Operational credentials — `ADMIN_TOKEN`, tenant API keys, database passwords, Vault credentials, Keycloak client secrets, and the Cardano Wallet passphrase — protect access to the services, and rotate through the same process used for the rest of your production secrets. DID keys are different: they are protocol-level material that verifiers depend on, and rotating them changes what the rest of the world can verify. The procedures below are based on Cloud Agent 2.1.0; check the release notes when you upgrade, because key-selection behavior is an area where the agent is still evolving.

**Operational credentials.** Most of these rotate with a restart, but a few have behavior worth knowing before you start:

| Credential | Procedure | Impact |
|---|---|---|
| `ADMIN_TOKEN` | Generate a new value, update the secret store, and restart the Cloud Agent. | The agent accepts one admin token. Admin automation fails until it uses the new value; tenant API calls are not affected. |
| Tenant API keys | Register a new key with `POST /iam/apikey-authentication`, move the tenant's application to it, then unregister the old key with `DELETE /iam/apikey-authentication` and the same `entityId`/`apiKey` body. | None, if done in that order: an entity can hold several API keys at once. |
| `API_KEY_SALT` | Do not rotate as routine maintenance. | The Cloud Agent stores API keys as salted hashes, so changing the salt invalidates every registered API key at once. Treat it as a re-provisioning event for all tenants. |
| Vault AppRole `secret_id` | See the runbook below. | None, if the old `secret_id` stays valid until every agent instance has restarted. |

Always generate a new random value for a replacement API key. The Cloud Agent keeps unregistered keys on record and refuses to register the same value again, and if a key that is already registered is submitted for a *different* entity, the agent treats it as compromised and disables it.

**Rotating the Vault AppRole `secret_id`.** The Cloud Agent logs in to Vault with `VAULT_APPROLE_ROLE_ID` and `VAULT_APPROLE_SECRET_ID` at startup, and logs in again with the same pair before each token lease expires. Revoking the old `secret_id` while an agent is still running on it does not fail immediately; it fails at the next re-login, when the agent can no longer read wallet seeds. Rotate in this order:

1. Generate a new `secret_id` for the role, and record its accessor:

   ```bash
   vault write -f auth/approle/role/cloud-agent/secret-id
   ```

2. Update `VAULT_APPROLE_SECRET_ID` in the production secret store and restart or roll every Cloud Agent instance.
3. Confirm each instance is healthy and can reach wallet material, for example by listing a tenant's DIDs.
4. Destroy the old `secret_id` by its accessor:

   ```bash
   vault write auth/approle/role/cloud-agent/secret-id-accessor/destroy \
     secret_id_accessor=replace-with-old-accessor
   ```

If the role sets `secret_id_ttl` or `secret_id_num_uses`, remember that the agent's periodic re-login consumes them. A `secret_id` that expires or runs out of uses while the agent is running has the same effect as revoking it.

**Rotating `did:prism` keys.** A `did:prism` controller rotates verification keys by publishing a signed DID **update** operation through PRISM Node, which changes the DID Document's verification methods and relationships on-chain (see [Chapter @sec-did-and-diddocuments]). In the Cloud Agent this is `POST /did-registrar/dids/{didRef}/updates`, with `ADD_KEY` and `REMOVE_KEY` actions. The agent derives the new key from the wallet seed and records its derivation path.

The important constraint is on the verifier side. When the Cloud Agent verifies a JWT credential, it resolves the issuer DID's *current* DID Document and checks the signature against the keys listed under `assertionMethod`. Removing a key therefore breaks verification of every credential that key signed, not just future ones. Rotate an issuing key with an overlap period:

1. **Add the new key.** Submit an update that adds a key with the same purpose and curve as the one being replaced:

   ```bash
   curl -X POST "https://agent.example.test/cloud-agent/did-registrar/dids/$ISSUER_DID/updates" \
     -H "apikey: replace-with-tenant-api-key" \
     -H "Content-Type: application/json" \
     -d '{
       "actions": [
         {
           "actionType": "ADD_KEY",
           "addKey": { "id": "assertion-2", "purpose": "assertionMethod", "curve": "secp256k1" }
         }
       ]
     }'
   ```

   The agent accepts one pending update per DID. Wait until the operation is confirmed on-chain and the DID resolves with the new key before submitting another update.

2. **Name the signing key in every issuance request.** While the DID has two `assertionMethod` keys, the Cloud Agent refuses to pick one: when it signs a JWT credential for an offer that did not set an issuing key ID, the issuance fails. The key is looked up at signing time, so this also catches offers created before the new key was added that are still waiting for the holder's request. Set `issuingKid` in `jwtVcPropertiesV1` (for example `"issuingKid": "assertion-2"`) on every offer, update your issuer application *before* step 1 is confirmed, and let in-flight offers complete first. OID4VCI issuance in this release does not take a key ID, so it fails for as long as the DID has more than one `assertionMethod` key; schedule the overlap accordingly if you use it.
3. **Reissue credentials** that must remain verifiable after the old key is removed, signing with the new key. Credentials that will expire or be revoked before the cutover can be left alone.
4. **Remove the old key** with a `REMOVE_KEY` action (`"removeKey": { "id": "assertion-1" }`) once no credential you still need depends on it. After removal, confirm that a freshly issued credential verifies, and that your status list credential still verifies. The agent signs status list credentials with the first `assertionMethod` key it finds, so a status list last signed with the removed key may need an update (a revocation or suspension) to be re-signed.

The DID's `master0` key, which signs the update operations themselves, is reserved: the Cloud Agent rejects `ADD_KEY` and `REMOVE_KEY` actions that name it, so it cannot be rotated through the API.

**The wallet seed cannot be rotated.** A wallet's seed is fixed when the wallet is created; the Cloud Agent has no endpoint to replace it, and every PRISM key in the wallet, including `master0`, is derived from it. "Rotating" a seed means creating a new wallet (with a new seed), creating and publishing new DIDs in it, reissuing credentials from the new DIDs, and moving tenants and relying parties over to them. Plan it as a migration.

That also defines the blast radius of a seed compromise. Anyone holding the seed can derive `master0` and publish update or deactivation operations for every PRISM DID in that wallet, so the response is urgent: deactivate the affected DIDs, create a new wallet and new DIDs, and reissue. Deactivating a DID means verifiers can no longer resolve usable keys for it, so every credential it issued stops verifying — which is the intended outcome when the signing keys can no longer be trusted, but should be communicated to holders and relying parties before it happens where possible. This is why the seed, and the Vault storage that holds it, is the single most important thing in the deployment to protect and back up.
