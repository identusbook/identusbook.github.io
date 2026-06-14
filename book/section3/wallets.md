# Wallets {#sec-wallets}

## Overview

An SSI wallet is software that stores and protects the material a holder, issuer, or verifier needs to act in an identity protocol. In the [W3C Verifiable Credentials Data Model 2.0](https://www.w3.org/TR/vc-data-model-2.0/), a credential repository stores and protects access to a holder's credentials. In [W3C DID Core](https://www.w3.org/TR/did-1.0/), a DID resolves to a DID document that can expose public verification methods and service endpoints. The private keys, credentials, DIDComm records, and local wallet state live in wallet storage, not in the DID document.

Identus uses the word wallet in two common places:

- An Edge Agent wallet runs inside a holder-facing application, such as a browser app, mobile app, or desktop app. The application embeds an Identus Edge SDK and provides storage through Pluto.
- A Cloud Agent wallet is a server-side wallet container accessed through the Cloud Agent REST API. A controller or tenant uses API calls and webhooks rather than direct in-process SDK calls.

Do not treat a ledger, PRISM Node, or DID resolver as a wallet backup. Public DID state can help other parties resolve issuer keys and service endpoints. It cannot reconstruct holder-held credentials, Peer DID private keys, DIDComm message history, connection records, or presentation state. A production wallet needs an explicit storage, encryption, backup, and recovery design.

For the airline ticket example in this section, the airline runs Cloud Agent as an issuer. The traveler uses an Edge Agent wallet as the holder. Airport security can use Cloud Agent, an Edge SDK, or another verifier implementation, depending on the deployment.

## Wallet Responsibilities

A wallet has more work to do than storing credentials. It owns the local state needed to continue protocol flows across app restarts, device changes, and network interruptions.

The wallet stores or references:

- DID records and DID metadata.
- Private keys, wallet seeds, or secret references used by DID operations.
- Verifiable Credentials received from issuers.
- Presentation records created for verifiers.
- DIDComm messages, connection records, and protocol state.
- Mediator configuration for offline delivery.
- Credential status and schema references needed by later verification flows.

The wallet uses that state to perform protocol work. The holder wallet receives credential offers, asks the holder for consent, stores issued credentials, creates presentations, and sends DIDComm responses. The issuer wallet signs credentials and tracks issuance records. The verifier wallet or verifier agent receives presentations, checks cryptographic proofs, resolves issuer material, checks credential status, and then applies verifier policy.

Keep the verification layers separate. A cryptographic check answers whether the presentation, credential proof, issuer key, holder binding, and status data verify. A policy check answers whether the verifier accepts that issuer, credential type, schema, subject, freshness, or assurance level for the business action.

## Edge Agent Wallets

Identus Edge Agent SDKs let an application run wallet and agent capabilities close to the holder. Current Identus repositories provide:

- [TypeScript SDK](https://github.com/hyperledger-identus/sdk-ts) for browser and Node.js applications. The [current TypeScript SDK docs](https://hyperledger-identus.github.io/docs/sdk-ts/docs/sdk/) name the package `@hyperledger/identus-sdk` and note the rename from `@hyperledger/identus-edge-agent-sdk`.
- [Swift SDK](https://github.com/hyperledger-identus/sdk-swift) for Apple platforms.
- [Kotlin Multiplatform SDK](https://github.com/hyperledger-identus/sdk-kmp) for Android and JVM applications.

The SDKs use the same architectural vocabulary:

- Apollo provides cryptographic operations.
- Castor creates, manages, and resolves DIDs.
- Pollux handles credential operations in the Swift and Kotlin SDKs. The current TypeScript SDK exposes credential capabilities through its modules and plugins.
- Mercury handles DIDComm v2 messages and related secure messaging work.
- Pluto provides the storage interface.
- Agent or EdgeAgent composes the building blocks into a higher-level agent API.

Start with Agent or EdgeAgent for application code. Direct calls into Apollo, Castor, Mercury, Pollux, or Pluto fit cases where the application needs a lower-level operation that the agent API does not expose. For example, a wallet that implements a custom DIDComm protocol can build and pack messages through Mercury, then store the resulting protocol records through the same wallet storage layer.

### Pluto Storage

Pluto is the key implementation boundary in an Edge Agent wallet. Identus defines the storage interface, and the application provides the storage implementation that matches its platform and threat model.

For a browser wallet, the implementation can use IndexedDB or another browser database. For a mobile wallet, it can use the platform database plus Keychain or Keystore-backed key protection. For a server-side test wallet, an in-memory store works for a disposable run. Production storage should encrypt sensitive data at rest, bind encryption keys to the platform security model where possible, and support tested backup and restore.

Storage design changes wallet behavior. If the application stores credentials but loses private keys, the holder can lose the ability to prove control of the DID used during issuance or presentation. If the application stores keys but loses credentials, the holder can still control the DID but no longer has the credential payload. If the application loses DIDComm protocol records, it can fail to resume issuance, connection, or proof flows after restart.

Treat community Pluto stores as dependencies to evaluate, not as Identus defaults. Check maintenance status, encryption behavior, migration support, and compatibility with the SDK version used by the application before selecting one.

## Cloud Agent Wallets

Cloud Agent runs on a server and exposes wallet operations through REST APIs and webhook-driven workflows. The [Cloud Agent README](https://github.com/hyperledger-identus/cloud-agent) describes it as a W3C and Aries standards-based cloud agent that supports [DIDComm v2](https://identity.foundation/didcomm-messaging/spec/v2.1/), credential issuance, credential verification, credential holding, secret management, and multi-tenancy.

A Cloud Agent wallet fits services that need server-side identity operations. An airline can use Cloud Agent to manage issuer DIDs, sign ticket credentials, send offers, and track issuance state. An airport verifier can use Cloud Agent to request presentations and receive verification results. A company can run Cloud Agent in multi-tenant mode when each customer or department needs a separate wallet.

Cloud Agent custody has a different trust boundary from an Edge Agent wallet. The tenant or business controller interacts with the wallet through an API. The service operator controls the runtime, database, secret storage, networking, and operational access. A service-managed wallet can be a good fit for organizations and delegated user wallets, but the deployment must define who can administer wallets, export data, rotate secrets, recover seeds, and inspect logs.

### Multi-Tenant Wallets

In Cloud Agent multi-tenancy, an administrator creates a wallet, creates an entity for the tenant, and provisions an authentication method for that entity. The [Identus tenant onboarding documentation](https://hyperledger-identus.github.io/docs/tutorials/multitenancy/tenant-onboarding/) describes the wallet as the container for tenant assets, including DIDs, credentials, and connections. After provisioning, Cloud Agent scopes tenant API calls to that wallet.

This model fits service-managed wallets. It does not remove custody risk. The administrator API, tenant authentication, wallet identifiers, database rows, Vault paths, and webhook routing must all preserve tenant separation.

### Cloud Agent Key Material

Identus Cloud Agent uses different key-management paths for PRISM DIDs and Peer DIDs.

For managed PRISM DIDs, Cloud Agent derives key material from a wallet seed and derivation path. The [DID management documentation](https://hyperledger-identus.github.io/docs/home/identus/cloud-agent/did-management/) states that Cloud Agent stores the derivation path rather than the PRISM key material itself, then reconstructs key material at runtime from the seed and path.

For managed Peer DIDs, Cloud Agent generates key material and stores it in secret storage. Peer DIDs support DIDComm activity, so these keys protect relationship-specific communications.

The [Cloud Agent secrets storage documentation](https://hyperledger-identus.github.io/docs/home/identus/cloud-agent/secrets-storage/) names HashiCorp Vault as the default secret storage service. It describes secret types such as seeds, private keys, credential-definition data, and AnonCreds link secrets. In multi-tenant configuration, Vault asset paths include the wallet ID so each wallet's secrets live under its own path.

## Choosing Edge or Cloud Custody

Use an Edge Agent wallet when the holder should keep credentials and keys in a user-controlled application. This model fits traveler wallets, employee wallets, citizen wallets, and mobile proof presentation. The application team must own storage, encryption, backup, recovery, mediator use, and local user consent.

Use a Cloud Agent wallet when a server must act for an organization or delegated tenant. This model fits issuers, verifiers, service-managed organizational wallets, and backend workflows that need REST APIs and webhooks. The deployment team must own secret storage, authentication, tenant separation, monitoring, and operational controls.

Many Identus systems use both. The issuer and verifier run Cloud Agent. The holder uses an Edge Agent wallet. DIDComm, credentials, DIDs, and presentation protocols connect the two custody models without requiring the same wallet implementation on every side.

## Airline Ticket Flow

The airline ticket example uses the wallet boundary in a concrete way:

1. The airline creates or selects a Cloud Agent wallet for issuer operations.
2. The airline creates an issuer `did:prism` with an `assertionMethod` verification method and publishes the DID when verifiers need ledger-backed resolution.
3. The traveler opens an Edge Agent wallet. The wallet creates or selects holder DIDs and stores local secret material through Pluto.
4. The airline sends a ticket credential offer to the traveler wallet through DIDComm or another supported issuance path.
5. The traveler wallet shows the offer, records holder consent, requests the credential, receives it, and stores it.
6. Airport security requests proof of ticket status, flight, passenger binding, or another accepted claim set.
7. The traveler wallet creates a presentation from the stored credential and sends it to the verifier.
8. The verifier software checks the presentation proof, issuer DID resolution, credential status, holder binding, schema, and format-specific rules.
9. Airport security policy decides whether the verified result satisfies the checkpoint rule.

Steps 8 and 9 are separate decisions. The verifier software can report that the credential proof is valid, the issuer DID resolved, and the credential is active. Airport policy can still reject the presentation if the flight date, airport, issuer, credential type, or passenger binding does not match the checkpoint rule.

## Implementation Notes

For Edge Agent wallets:

- Pick the SDK for the host platform before designing storage.
- Treat Pluto as a required wallet component, not an optional cache.
- Encrypt wallet contents that include credentials, DIDComm records, seeds, private keys, or secret references.
- Test restore on a clean device or browser profile.
- Use `did:peer` for relationship-specific DIDComm activity and `did:prism` for public resolution.

For Cloud Agent wallets:

- Decide whether the deployment uses a default wallet or multi-tenancy before onboarding users.
- Keep administrator credentials separate from tenant credentials.
- Place wallet seeds and private key material in the configured secret storage.
- Back up the Cloud Agent database and secret storage as one recovery unit.
- Audit webhook delivery, tenant API keys, wallet IDs, and Vault paths during tenant onboarding.

A team completes wallet work only after it tests restore and protocol resumption. A wallet that can issue or receive credentials during a happy-path demo can still fail in production if the holder changes devices, the browser storage is cleared, the mediator queue grows, or a Cloud Agent tenant needs recovery after database restore.
