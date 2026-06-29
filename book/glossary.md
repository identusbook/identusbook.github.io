# Glossary {#sec-glossary .unnumbered}

This glossary collects the Self-Sovereign Identity (SSI), decentralized-identity, and Identus-specific terms used throughout the book. Definitions are written for the working developer: enough to follow the surrounding chapters, with pointers to the standards where a term is formally defined. Terms are listed alphabetically.

[ADA]{#ada}

:	The native cryptocurrency of the [Cardano](#cardano) blockchain. PRISM Node spends ADA on transaction fees when it anchors `did:prism` operations on-chain — test ADA on test networks such as `preprod`, real ADA on `mainnet`.

[AnonCreds (Anonymous Credentials)]{#anoncreds}

:	A privacy-preserving [Verifiable Credential](#verifiable-credential) format originating in the Hyperledger Indy ecosystem. It uses [zero-knowledge proofs](#zero-knowledge-proof) and [predicate proofs](#predicate-proof) so a holder can prove facts (for example, "age over 21") without revealing the underlying attribute values. Supported by the Identus Cloud Agent.

[API key]{#api-key}

:	A per-[tenant](#tenant) credential (sent in an `apikey` header) that scopes Cloud Agent API calls to a specific tenant's [wallet](#wallet). Privileged administrative operations instead use an admin token.

[Apollo]{#apollo}

:	The Identus [building block](#building-blocks) that provides cryptographic primitives — hashing, digital signatures, encryption, and key generation — used across the platform to ensure data integrity, authenticity, and confidentiality.

[AppRole]{#approle}

:	A [HashiCorp Vault](#hashicorp-vault) authentication method (a role ID plus a secret ID) preferred over a static root token for automated, production Cloud Agent deployments.

[Aries (Hyperledger Aries)]{#aries}

:	A family of Hyperledger interoperability protocols (RFCs) for SSI agents — covering DID exchange, out-of-band invitations, issue-credential, and present-proof flows. The Identus Cloud Agent is built to be W3C- and Aries-compatible.

[assertionMethod]{#assertion-method}

:	A [verification relationship](#verification-relationship) in a [DID Document](#did-document) listing the keys approved for making assertions, including signing (issuing) Verifiable Credentials. A verifier checks that the credential's signing key appears here.

[authentication]{#authentication}

:	A [verification relationship](#verification-relationship) in a [DID Document](#did-document) listing the keys approved for proving control of the DID through challenge-response authentication.

[Building Blocks]{#building-blocks}

:	The set of focused, modular Identus libraries that each handle one area of SSI functionality: [Apollo](#apollo) (cryptography), [Castor](#castor) (DIDs), [Pollux](#pollux) (credentials), [Mercury](#mercury) (DIDComm), and [Pluto](#pluto) (storage). Both the Cloud Agent and the Edge Agent SDKs implement the same building blocks.

[Cardano]{#cardano}

:	The public blockchain Identus uses as its production [Verifiable Data Registry](#vdr), anchoring `did:prism` operations so that anyone can resolve a published PRISM DID. Anchoring requires [ADA](#ada) to pay transaction fees.

[Cardano DB Sync]{#cardano-db-sync}

:	A Cardano component that follows the chain and indexes blocks into a PostgreSQL database so that PRISM Node can read ledger data efficiently. Typically the heaviest resource consumer in a self-hosted Cardano stack.

[Cardano Wallet]{#cardano-wallet}

:	The Cardano-ecosystem service PRISM Node uses to manage funds and submit Cardano transactions when publishing DID operations. Distinct from an SSI [wallet](#wallet).

[Castor]{#castor}

:	The Identus [building block](#building-blocks) that creates, manages, and resolves [Decentralized Identifiers](#did), currently supporting the `did:prism` and `did:peer` methods.

[Certificate Authority (CA)]{#certificate-authority}

:	The traditional centralized model of digital trust, in which a central authority vouches for identities. Contrasted in this book with a [Trust Registry](#trust-registry), which records authority within a governed ecosystem but does not itself issue or hold credentials.

[Claim]{#claim}

:	A single statement or attribute about a [subject](#subject) within a credential — for example a name, a date of birth, or a license category. A [Verifiable Credential](#verifiable-credential) is a set of claims made by an [issuer](#issuer).

[Cloud Agent]{#cloud-agent}

:	The server-side Identus agent (written in Scala) that manages wallets, DIDs, and DIDComm messaging, and that can issue, hold, and verify Verifiable Credentials. It is driven by a [controller](#controller) application through its REST API and is expected to be online at all times.

[cnf (confirmation key)]{#cnf}

:	A key-binding element embedded in an [SD-JWT](#sd-jwt) credential. It binds the credential to a holder key so the holder can later prove control by signing the verifier's [challenge](#challenge) and [domain](#domain).

[Confirmation depth]{#confirmation-depth}

:	The number of blocks that must be added after a DID transaction before its state is treated as final and safe to resolve. A tuning knob when anchoring DIDs on a ledger.

[Connection]{#connection}

:	An established, secure, ongoing [DIDComm](#didcomm) relationship between two agents, typically underpinned by [Peer DIDs](#peer-did) and tracked as a connection record in each party's wallet.

[Controller (application)]{#controller}

:	The application that drives a [Cloud Agent](#cloud-agent) through its REST API and webhook callbacks. The controller holds the business logic; the agent handles the SSI standards work. (Not to be confused with a [DID controller](#did-controller).)

[Coordinate Mediation]{#coordinate-mediation}

:	The DIDComm protocol (`coordinate-mediation/2.0`) a wallet uses to request [mediation](#mediation) from a [mediator](#mediator) and register the [recipient keys](#recipient-keys) for which the mediator should accept messages.

[Correlation]{#correlation}

:	The linking of a person's or organization's activity across different contexts by means of a shared identifier or reused key. Public DIDs and reused DID Document material create correlation risk; [pairwise DIDs](#pairwise-did) reduce it.

[COSE]{#cose}

:	The CBOR-based family of cryptographic standards for signing and encryption — the binary counterpart to [JOSE](#jose) — referenced by the W3C VC "Securing with JOSE/COSE" recommendation.

[Credential offer]{#credential-offer}

:	A message from an [issuer](#issuer) proposing to issue a specific credential to a [holder](#holder), which begins the [credential issuance](#credential-issuance) flow.

[Credential issuance]{#credential-issuance}

:	The protocol flow by which an issuer creates, signs, and delivers a [Verifiable Credential](#verifiable-credential) into a holder's wallet. In Identus this follows the Issue Credential protocol over DIDComm, or [OpenID4VCI](#openid4vci).

[Credential schema]{#credential-schema}

:	A registered, shared definition of a credential type's structure — its fields, types, and required attributes. Issuers issue against it, and verifiers and [verification policies](#verification-policy) are scoped to it. Identus supports JSON Schema (for JWT/SD-JWT) and AnonCreds schemas.

[credentialStatus]{#credential-status}

:	The property inside a revocable credential that points to its [status list](#status-list) entry, allowing a verifier to discover whether the credential has been [revoked or suspended](#revocation). Defined by the W3C VC Data Model.

[Decentralized Identifier (DID)]{#did}

:	A globally unique identifier that an entity controls without a central registrar. Defined by the [W3C](#w3c) DID Core specification, a DID is a URI with three parts — the `did:` scheme, a [method](#did-method) name, and a method-specific identifier — and it resolves to a [DID Document](#did-document).

[Derivation path]{#derivation-path}

:	The deterministic path used together with a [wallet seed](#wallet-seed) to regenerate a specific DID's keys (hierarchical-deterministic key derivation). The Cloud Agent stores the path rather than the derived keys.

[DID controller]{#did-controller}

:	The entity authorized, under a DID method's rules, to change a [DID Document](#did-document) (rotate keys, add services, deactivate). The controller may be the same as the [DID subject](#did-subject) or a different entity acting on its behalf.

[DID Document]{#did-document}

:	The document a DID resolves to. It publishes the public material needed to interact with the DID subject — [verification methods](#verification-method), [verification relationships](#verification-relationship), and [service endpoints](#service-endpoint). It should never contain private keys, secrets, or a personal profile.

[DID method]{#did-method}

:	The scheme (the part after `did:`, such as `prism` or `peer`) that defines how a particular class of DID is created, resolved, updated, and deactivated. The W3C defines the shared data model; each method specification defines its own registry, operations, and lifecycle.

[`did:peer`]{#did-peer}

:	See [Peer DID](#peer-did).

[`did:prism`]{#did-prism}

:	See [PRISM DID](#prism-did).

[DID resolution]{#did-resolution}

:	The process of looking up a DID and returning its [DID Document](#did-document) plus resolution metadata. Performed by a [resolver](#resolver) that understands the relevant DID method.

[DIDComm (DIDComm Messaging V2)]{#didcomm}

:	A secure, encrypted, transport-agnostic messaging protocol for communication between DID-identified agents. DIDComm V2 (the version Identus uses) hides message content from everyone except authorized recipients, proves the sender in authenticated mode, and works over any transport (HTTP, WebSockets, Bluetooth, and so on).

[DIDPair]{#didpair}

:	The cryptographic pairing of a sender DID and a recipient DID that together identify a DIDComm channel. A [mediator](#mediator) maintains a message queue per DIDPair.

[DIF (Decentralized Identity Foundation)]{#dif}

:	An industry organization that develops decentralized-identity standards, including DIDComm and Presentation Exchange.

[DIF Presentation Exchange]{#presentation-exchange}

:	A DIF specification for expressing what proof a verifier requires and how a holder's wallet responds. The Identus Cloud Agent implements it for credential requests and submissions.

[Disclosure (SD-JWT)]{#disclosure}

:	In an [SD-JWT](#sd-jwt) credential, the salted claim value a holder reveals at presentation time so the verifier can hash it and match it against the signed digest in the credential. Undisclosed claims stay hidden.

[Edge Agent]{#edge-agent}

:	An Identus agent that runs inside a user-facing application (web, mobile, desktop), implemented as an [SDK](#edge-sdk). Unlike a [Cloud Agent](#cloud-agent), an Edge Agent cannot be assumed to be online and relies on a [mediator](#mediator) to send and receive messages. Edge Agents typically act as the user's [wallet](#wallet).

[Edge SDK]{#edge-sdk}

:	The Identus client libraries — available in TypeScript, Swift, and Kotlin Multiplatform — that embed [Edge Agent](#edge-agent) capabilities into an application. They implement the same [building blocks](#building-blocks) as the Cloud Agent.

[Ecosystem]{#ecosystem}

:	A community of participants operating under a shared [governance framework](#governance-framework) within which trust relationships and authorizations are defined. A [Trust Registry](#trust-registry) answers questions about authority within a specific ecosystem.

[Ed25519]{#ed25519}

:	A widely used elliptic-curve digital-signature algorithm and key type, used for authentication and credential signing (for example, required for SD-JWT issuer and holder keys in Identus).

[Entity]{#entity}

:	In the Cloud Agent, the administrative object that represents a [tenant](#tenant) and is bound to a [wallet](#wallet). Authentication (an [API key](#api-key) or Keycloak permission) is attached to an entity. In single-tenant mode a default entity (ID all-zeros) is used automatically.

[`forward` message]{#forward-message}

:	A DIDComm routing message (from the `routing/2.0` protocol) that wraps an already-encrypted message for a final recipient inside an outer envelope addressed to a [mediator](#mediator), so the mediator can route it without reading the inner content.

[Governance authority]{#governance-authority}

:	The body that defines an [ecosystem's](#ecosystem) rules and maintains, or oversees, its [Trust Registry](#trust-registry).

[Governance framework]{#governance-framework}

:	The set of rules, published by a [governance authority](#governance-authority), describing who may do what within an [ecosystem](#ecosystem). A [Trust Registry](#trust-registry) is its runtime, queryable expression. Authority comes from the governance framework, not from the registry technology itself.

[HashiCorp Vault]{#hashicorp-vault}

:	A secrets-management service that the Cloud Agent can use as its production [secret storage](#secret-storage) backend for wallet seeds and key material, with per-wallet paths in multi-tenant deployments.

[Holder]{#holder}

:	The role that receives [Verifiable Credentials](#verifiable-credential), stores them (usually in a [wallet](#wallet)), and creates [presentations](#verifiable-presentation) from them in response to verifier requests. The holder is often, but not always, the [subject](#subject) of the credential.

[Holder binding]{#holder-binding}

:	Cryptographic evidence that the party presenting a credential actually controls the key associated with the credential's subject. See also [key binding](#key-binding).

[Hyperledger Indy]{#hyperledger-indy}

:	A Hyperledger blockchain/SSI ecosystem from which the [AnonCreds](#anoncreds) credential format originated.

[Identus (Hyperledger Identus)]{#identus}

:	The open-source decentralized-identity platform this book is about, hosted under the Linux Foundation's decentralized-trust umbrella. It provides the [Cloud Agent](#cloud-agent), [PRISM Node](#prism-node), [Edge SDKs](#edge-sdk), and [Mediator](#mediator) for issuing, holding, and verifying credentials.

[Issue Credential protocol]{#issue-credential}

:	The DIDComm protocol (Issue Credential 3.0 in Identus) that governs the credential offer, request, and issuance message exchange between issuer and holder.

[Issuer]{#issuer}

:	The role that makes [claims](#claim) about a [subject](#subject), packages them as a [Verifiable Credential](#verifiable-credential), and cryptographically signs it. The issuer's authority comes from the surrounding context — law, accreditation, contract, or governance.

[Issuer DID]{#issuer-did}

:	A DID controlled by an [issuer](#issuer) and used to sign the credentials it issues, so verifiers can resolve the issuer's keys and check authorship. Issuer DIDs are typically public ([PRISM DIDs](#prism-did)).

[JOSE]{#jose}

:	The JSON-based family of cryptographic standards (JSON Web Signature, JSON Web Encryption, JSON Web Key, JSON Web Token) commonly used to secure credentials. See also [COSE](#cose).

[JSON Web Key (JWK)]{#jwk}

:	A JSON format for representing a cryptographic key. DID Documents and DIDComm key material are often expressed as JWKs.

[JWT-VC]{#jwt-vc}

:	A [Verifiable Credential](#verifiable-credential) format that packages claims inside a signed JSON Web Token (JWT). A JWT-VC is generally presented whole, so all of its claims are revealed (no [selective disclosure](#selective-disclosure)).

[keyAgreement]{#key-agreement}

:	A [verification relationship](#verification-relationship) in a [DID Document](#did-document) listing keys approved for key agreement — for example, establishing the encryption used by [DIDComm](#didcomm). Typically backed by an [X25519](#x25519) key.

[Key binding]{#key-binding}

:	Cryptographically tying a credential to a holder's key so the holder can prove control of that key when presenting the credential. See also [holder binding](#holder-binding) and [cnf](#cnf).

[Keycloak]{#keycloak}

:	An identity and access-management server that Identus can integrate so that [tenants](#tenant) authenticate via [OIDC](#oidc) and receive wallet permissions via [UMA](#uma), replacing static API keys.

[Ledger anchoring]{#ledger-anchoring}

:	Recording DID operations as transactions on a blockchain (such as [Cardano](#cardano)) so they become tamper-evident and publicly resolvable.

[Link secret]{#link-secret}

:	A holder-controlled secret used by [AnonCreds](#anoncreds) to bind issued credentials to the holder and to enable zero-knowledge proofs across multiple credentials.

[Long-form / Short-form PRISM DID]{#long-short-form}

:	Two encodings of a [PRISM DID](#prism-did). The long form embeds the DID's full initial state in the identifier and is resolvable before the DID is published. The short form is the compact identifier used once the DID has been [published](#published-did) to the ledger.

[Mediation]{#mediation}

:	The arrangement a holder's wallet establishes with a [mediator](#mediator) so the mediator will receive and relay DIDComm messages on its behalf. It produces routing information the wallet then advertises in its DID Document.

[Mediator]{#mediator}

:	A DIDComm V2 service that acts as a stable, always-online relay for agents — such as mobile wallets — that are not continuously reachable. It queues encrypted messages per [DIDPair](#didpair) and delivers them when the recipient polls or connects, without being able to read the encrypted content. It is *not* a [VDR](#vdr).

[Mercury]{#mercury}

:	The Identus [building block](#building-blocks) that provides the [DIDComm](#didcomm) V2 interface for secure agent-to-agent messaging, independent of the underlying transport.

[Message Pickup]{#message-pickup}

:	The DIDComm protocol (`messagepickup/3.0`) a wallet uses to poll a [mediator](#mediator) for queued messages, request their delivery, and confirm receipt.

[Multi-tenancy]{#multi-tenancy}

:	A [Cloud Agent](#cloud-agent) mode in which a single agent instance hosts many isolated [tenants](#tenant), each with its own [wallet](#wallet), DIDs, credentials, and connections. The opposite is single-tenant mode, which serves one default wallet.

[NeoPRISM]{#neoprism}

:	A newer DID-node backend for Identus, recommended for production, that can replace the legacy [PRISM Node](#prism-node) as the way `did:prism` operations are published and resolved.

[OpenID4VCI (OID4VCI)]{#openid4vci}

:	OpenID for Verifiable Credential Issuance — an OAuth 2.0-based protocol for a wallet to obtain credentials from an issuer's credential endpoint. An alternative issuance path to DIDComm.

[OpenID4VP]{#openid4vp}

:	OpenID for Verifiable Presentations — an OAuth 2.0-based protocol for requesting and presenting credentials. An alternative presentation path to DIDComm.

[OIDC (OpenID Connect)]{#oidc}

:	A standard authentication protocol layered on OAuth 2.0, used (via [Keycloak](#keycloak)) to authenticate tenants and issue access tokens.

[Out-of-Band (OOB) invitation]{#oob-invitation}

:	A self-contained DIDComm invitation — often delivered as a URL or QR code (carried in an `_oob` query parameter) — that lets a new party start a [connection](#connection), [mediation](#mediation), or a connectionless issuance/presentation without a pre-existing channel.

[Pairwise DID]{#pairwise-did}

:	A DID created uniquely for a single relationship, so that each relationship uses a different pseudonymous identifier. Pairwise (relationship-specific) DIDs reduce [correlation](#correlation) across contexts; [Peer DIDs](#peer-did) are typically used this way.

[Peer DID]{#peer-did}

:	A DID (method `did:peer`) shared privately between the parties to a relationship and not published to any public ledger. Identus uses Peer DIDs for [DIDComm](#didcomm) connections and [mediation](#mediation), where relationship-specific identifiers reduce correlation.

[Pluto]{#pluto}

:	The Identus [building block](#building-blocks) that defines the storage interface for identity data (DIDs, keys, credentials, connection state). The application supplies the concrete storage implementation.

[Pollux]{#pollux}

:	The Identus [building block](#building-blocks) (and Cloud Agent subsystem) responsible for [Verifiable Credential](#verifiable-credential) operations — issuance, verification, [selective disclosure](#selective-disclosure), and [credential status](#status-list).

[Predicate proof]{#predicate-proof}

:	A proof that an attribute satisfies a condition (for example, `age >= 21`) without disclosing the underlying value. A core feature of [AnonCreds](#anoncreds) and [zero-knowledge proofs](#zero-knowledge-proof).

[Presentation request]{#presentation-request}

:	A message (also called a proof request) in which a [verifier](#verifier) specifies exactly what proof it needs from a holder. Carried by the Present Proof protocol.

[Present Proof protocol]{#present-proof}

:	The DIDComm, format-neutral protocol (Present Proof 3.0 in Identus) that carries a [presentation request](#presentation-request) from a verifier and a [presentation](#verifiable-presentation) back from a holder.

[PRISM DID]{#prism-did}

:	A DID using Identus's `did:prism` method, whose create, update, and deactivate operations are anchored to the [Cardano](#cardano) ledger via [PRISM Node](#prism-node). PRISM DIDs suit public issuer and verifier identities that need ledger-backed resolution. See also [long-form / short-form](#long-short-form).

[PRISM Node]{#prism-node}

:	The Identus component that implements the `did:prism` method, acting as a second-layer node over the ledger. It publishes and resolves PRISM DIDs, maintaining an indexed internal state synchronized with the underlying blockchain. It serves as the [VDR](#vdr) for PRISM DIDs and is expected to be online at all times.

[Published DID]{#published-did}

:	A DID whose operations have been anchored to the ledger so that any party can resolve it. Contrasted with an unpublished DID — for example, a holder usually does not need to publish their DID, whereas an issuer typically must.

[Recipient keys]{#recipient-keys}

:	The keys a wallet registers with a [mediator](#mediator) so the mediator knows which incoming messages to queue for that recipient.

[Relying party]{#relying-party}

:	The party that consumes a verification result and applies business and trust policy — accepted issuers, schemas, and credential types — to decide whether to act. Often the same actor as the [verifier](#verifier).

[Resolver]{#resolver}

:	Software that performs [DID resolution](#did-resolution): given a DID, it returns the [DID Document](#did-document). A [Universal Resolver](#universal-resolver) routes many methods to method-specific drivers. Resolver choice is itself a trust decision.

[Revocation / Suspension]{#revocation}

:	Invalidating a previously issued credential — permanently (revocation) or temporarily (suspension) — before its natural expiry. The change is reflected through a [status list](#status-list) that verifiers check via [credentialStatus](#credential-status).

[SD-JWT / SD-JWT-VC]{#sd-jwt}

:	Selective Disclosure JWT — a JWT variant that uses salted hashes (digests) so a holder can reveal only selected claims while the verifier checks each disclosed value against the signed digest. SD-JWT-VC is the Verifiable Credential format built on it. Supported by Identus. See [disclosure](#disclosure) and [cnf](#cnf).

[secp256k1]{#secp256k1}

:	An elliptic-curve key and signature algorithm (also used by Bitcoin and Cardano) supported by Identus for certain credential and status-list proofs.

[Secret storage]{#secret-storage}

:	The Cloud Agent's configurable store for wallet seeds and private keys. A `postgres` backend stores secrets in plaintext (lab use only); a [HashiCorp Vault](#hashicorp-vault) backend is recommended for production.

[Selective disclosure]{#selective-disclosure}

:	A holder's ability to reveal only the specific claims a verifier needs, rather than an entire credential — for example, proving "over 21" without revealing a full birth date. It depends on the credential format ([SD-JWT](#sd-jwt) and [AnonCreds](#anoncreds) support it; plain [JWT-VC](#jwt-vc) does not) and supports the practice of data minimization.

[Self-Sovereign Identity (SSI)]{#ssi}

:	A model of digital identity built around user-held credentials and cryptographic proofs rather than a central identity provider. A subject receives a credential from an [issuer](#issuer), stores it as a [holder](#holder), and presents proof to a [verifier](#verifier) who checks it against its own policy. SSI changes the custody and exchange of identity data; governance, law, and reputation still shape trust.

[Service endpoint]{#service-endpoint}

:	An entry in a [DID Document](#did-document) advertising where and how to interact with the DID subject — for example, a `DIDCommMessaging` endpoint that names the URL, accepted DIDComm profile, and routing keys.

[Status list]{#status-list}

:	A published, space-efficient revocation mechanism (such as a Bitstring Status List / StatusList2021) in which each bit represents the [revocation or suspension](#revocation) status of one credential. A verifier reads the bit referenced by a credential's [credentialStatus](#credential-status). In Identus this is provided by [Pollux](#pollux).

[Subject]{#subject}

:	The person, organization, device, account, or thing that a credential's [claims](#claim) describe. The subject is often the [holder](#holder) but need not be — a parent may hold a credential about a child, or a fleet manager about a device.

[Tenant]{#tenant}

:	A logical user or organization served by a [Cloud Agent](#cloud-agent), represented by an [entity](#entity) and backed by an isolated [wallet](#wallet). See [multi-tenancy](#multi-tenancy).

[ToIP (Trust over IP Foundation)]{#toip}

:	A Linux Foundation body developing standards for decentralized digital trust, including the ToIP Stack (a layered model of DIDs, DIDComm, data-exchange protocols, and application ecosystems) and the [Trust Registry Query Protocol](#trqp).

[Triangle of Trust]{#triangle-of-trust}

:	A common way of describing the three SSI roles — [issuer](#issuer), [holder](#holder), and [verifier](#verifier) — and the credential/presentation flows between them. The roles are interaction-specific: the same entity can issue one credential, hold another, and verify a third.

[TRQP (Trust Registry Query Protocol)]{#trqp}

:	A [ToIP](#toip) read-only protocol for asking a [Trust Registry](#trust-registry) whether an entity holds a given authorization under a given ecosystem's governance — in plain terms, "Does entity X hold authorization Y under governance framework Z?" Sometimes described as "DNS for trust registries."

[Trust Registry]{#trust-registry}

:	An authoritative, governed, machine-readable record of which entities are authorized to play which roles in an [ecosystem](#ecosystem). A verifier queries it to confirm an issuer's *authority* — a separate question from the cryptographic *validity* of a credential. A Trust Registry is not a [Certificate Authority](#certificate-authority) and is not a [VDR](#vdr).

[Trusted issuers]{#trusted-issuers}

:	The set of [issuer DIDs](#issuer-did), usually scoped to a [credential schema](#credential-schema), that a verifier accepts as authoritative. In Identus this is configured today in a [verification policy](#verification-policy) and is the concept a [Trust Registry](#trust-registry) generalizes.

[UMA (User-Managed Access)]{#uma}

:	An OAuth 2.0-based authorization standard that [Keycloak](#keycloak) uses to grant a subject permission to a specific Cloud Agent [wallet](#wallet) resource, issuing a Requesting Party Token (RPT) the tenant presents to the agent.

[Universal Resolver]{#universal-resolver}

:	A public [resolver](#resolver) service able to resolve many different [DID methods](#did-method) by routing each to a method-specific driver.

[Validation]{#validation}

:	The verifier's business-rule check on whether a credential is *appropriate* for a particular decision — for example, whether the issuer is acceptable under a hiring policy. Distinct from [verification](#verification): a credential can be cryptographically valid yet fail validation.

[Verifiable Credential (VC)]{#verifiable-credential}

:	A set of [claims](#claim) made by an [issuer](#issuer), secured with a cryptographic mechanism so that software can detect tampering and confirm authorship. Defined by the [W3C](#w3c) Verifiable Credentials Data Model. The data model is separate from the credential format (see [JWT-VC](#jwt-vc), [SD-JWT-VC](#sd-jwt), [AnonCreds](#anoncreds)).

[Verifiable Data Registry (VDR)]{#vdr}

:	A system — often a blockchain, but potentially a database, distributed network, or in-memory store — against which DIDs and related data are published and resolved, so that independent parties can verify authenticity and integrity without a central authority. Identus accesses VDRs through pluggable drivers (for example, the [PRISM Node](#prism-node) and [NeoPRISM](#neoprism) drivers). A [mediator](#mediator) is not a VDR.

[Verifiable Presentation (VP)]{#verifiable-presentation}

:	Data derived from one or more [Verifiable Credentials](#verifiable-credential) and shared with a specific [verifier](#verifier) in response to a request. Depending on the credential format, a presentation can include a whole credential, parts of one (see [selective disclosure](#selective-disclosure)), or data from several.

[Verification]{#verification}

:	The technical check that a credential or presentation is cryptographically sound: valid signatures, correctly resolved issuer keys, proper key usage ([verification relationships](#verification-relationship)), current [status](#status-list), and [holder binding](#holder-binding). Distinct from [validation](#validation), which applies business policy.

[Verification method]{#verification-method}

:	An entry in a [DID Document](#did-document) that specifies a public key (for example as a [JWK](#jwk)), its type, and its controller. Verification methods are referenced by [verification relationships](#verification-relationship) for specific purposes.

[Verification policy]{#verification-policy}

:	The verifier-side configuration in Identus expressing which [credential schemas](#credential-schema) and [trusted issuers](#trusted-issuers) are accepted. Applied after cryptographic checks, it is the practical form of acceptance policy / [validation](#validation).

[Verification relationship]{#verification-relationship}

:	The association in a [DID Document](#did-document) between a [verification method](#verification-method) and a permitted purpose — [authentication](#authentication), [assertionMethod](#assertion-method), [keyAgreement](#key-agreement), `capabilityInvocation`, or `capabilityDelegation`. These relationships prevent a key from being reused across proof purposes.

[Verifier]{#verifier}

:	The role that requests a [presentation](#verifiable-presentation) from a holder and decides whether to accept it, performing both [verification](#verification) (cryptographic checks) and [validation](#validation) (business policy). See also [relying party](#relying-party).

[Wallet]{#wallet}

:	In SSI, software that stores and protects a party's DIDs, keys, [connections](#connection), and credentials, and that creates presentations. In the Identus Cloud Agent, a wallet is also the unit of [tenant](#tenant) isolation. A holder's wallet is usually an [Edge Agent](#edge-agent); a server-side wallet runs in a [Cloud Agent](#cloud-agent). Distinct from a [Cardano Wallet](#cardano-wallet).

[Wallet seed]{#wallet-seed}

:	The secret root key material from which a wallet deterministically derives its DID keys, via a stored [derivation path](#derivation-path). It is a high-value secret: losing it can permanently break the ability to use or update existing DIDs.

[Webhook]{#webhook}

:	The HTTP callback mechanism by which the [Cloud Agent](#cloud-agent) notifies a [controller](#controller) of state changes — a received connection, a credential issued or received, a presentation verified, and so on.

[X25519]{#x25519}

:	An elliptic-curve key type used for Diffie-Hellman key agreement to set up the encryption used by [DIDComm](#didcomm). Appears in DID Documents under the [keyAgreement](#key-agreement) relationship.

[Zero-knowledge proof (ZKP)]{#zero-knowledge-proof}

:	A cryptographic technique that proves a statement is true without revealing the underlying data. The basis of [AnonCreds](#anoncreds) [predicate proofs](#predicate-proof) such as "over 21" without disclosing a birth date.
