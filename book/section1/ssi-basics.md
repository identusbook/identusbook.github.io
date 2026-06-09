# SSI Basics {#sec-ssi-basics}

## What SSI Means in This Book

Self-Sovereign Identity (SSI) describes identity systems built around user-held credentials and verifiable proofs, rather than a single identity provider that stores the user's account profile. A person, organization, device, or software agent can receive a credential from an issuer, store it in a wallet or agent, and present proof to a verifier. The verifier checks the proof and applies its own policy without a custom account integration with every issuer.

SSI does not make holder-provided claims trustworthy by itself. A verifier checks who issued the credential, what the credential says, whether the proof is valid, whether the credential is still in good standing, and whether the issuer is acceptable for that transaction. SSI changes custody and exchange of identity data; governance, law, contracts, and reputation still shape trust decisions.

This chapter uses standard terms from the [W3C Decentralized Identifiers (DIDs) v1.0 Recommendation](https://www.w3.org/TR/did-1.0/) and the [W3C Verifiable Credentials Data Model v2.0 Recommendation](https://www.w3.org/TR/vc-data-model-2.0/). DID Core defines DIDs as identifiers for verifiable, decentralized digital identity that can be decoupled from centralized registries, identity providers, and certificate authorities. The VC Data Model defines credentials, presentations, issuers, holders, subjects, and verifiers. The [Identus Quick Start Guide](https://hyperledger-identus.github.io/docs/home/quick-start/) describes Identus as core libraries for SSI interactions between issuers, holders, and verifiers.

## The SSI Interaction

A typical exchange has four steps.

1. An issuer makes claims about a subject and packages those claims as a credential.
2. A holder receives the credential and stores it in a wallet or agent.
3. A verifier asks the holder for proof that satisfies a specific request.
4. The holder returns a presentation, and the verifier checks cryptography, status, and business policy.

Example: a university issues a degree credential to Alice. Alice receives the credential in her wallet and acts as holder. The degree describes Alice, making Alice the subject. An employer acts as verifier and requests proof that Alice has a computer science degree from an accredited university. Alice's wallet creates a presentation from the credential. The employer's software checks that the university signed the credential, that Alice can prove control of the holder keys required by the credential format, that the issuer has not suspended or revoked the credential, and that the university is accepted under the employer's hiring policy.

Policy evaluation is validation. Cryptographic verification can show that an issuer's key secured the credential and that no one modified the credential after issuance. It cannot show that the issuer is acceptable for a particular decision. A diploma from an unknown training provider can pass cryptographic verification and still fail an employer's policy. The W3C VC specification describes validation as the verifier's business-rule check for whether a credential is proper for a particular use.

## DIDs and DID Documents

A Decentralized Identifier (DID) is a URI. It has a `did:` scheme, a DID method, and a method-specific identifier. In `did:prism:4a9bce8d72e4c30017c42f2b`, the method is `prism`; the last segment is the identifier data defined by that method.

The DID method tells software how to create, resolve, update, and deactivate that class of DID. DID Core does not require one storage technology. A DID method can use a distributed ledger, a database, a peer-to-peer network, or another verifiable data registry, as long as the method defines the required operations.

A DID resolves to a DID Document. DID Documents express public verification material, verification relationships, and services. A DID Document can tell software which public keys can be used for authentication, assertion, key agreement, capability invocation, or capability delegation. It can list service endpoints for communication protocols such as DIDComm. It should not contain secrets, private keys, or a personal profile.

The DID subject and DID controller may be the same entity, but they do not have to be. The subject is the person, organization, device, data model, or other thing identified by the DID. The controller is the entity that can change the DID Document according to the DID method. A company DID might identify the company as subject, and the company might delegate key operations to a Cloud Agent it controls.

DIDs can create correlation risk. DID Core warns that globally unambiguous identifiers can link activity across contexts. Pairwise DIDs reduce that risk by giving each relationship its own pseudonymous DID. Apply the same privacy review to DID Documents: reused keys, unusual service endpoints, or descriptive properties can still link activity across relationships.

## Verifiable Credentials

A credential is a set of claims made by an issuer. A Verifiable Credential (VC) is a credential with a securing mechanism that lets software detect tampering and check authorship. The VC can be about a person, organization, device, account, shipment, software artifact, or any other subject.

The issuer decides what to claim and signs or otherwise secures the credential. The holder stores the credential. The subject is the entity the claims describe. The holder and subject often match, but not always. A parent can hold a credential about a child. A company officer can hold a credential about the company. A device fleet manager can hold credentials about devices.

The data model is separate from the credential format. W3C VC 2.0 defines the model. Implementations can secure and transport credentials through different formats and protocols. Identus chapters later in the book use JWT-VC, SD-JWT-VC, and AnonCreds in implementation flows, so the right choice depends on the credential formats supported by the agents and wallets in that flow.

Credential status is part of production verification. W3C VC 2.0 defines `credentialStatus` for discovering status information such as suspension or revocation. The details depend on the status method. A verifier that accepts a credential without checking status may accept a credential the issuer no longer stands behind.

## Verifiable Presentations

A Verifiable Presentation (VP) is data derived from one or more credentials and shared with a specific verifier. A presentation can include one credential, parts of one credential, or data from more than one credential, depending on the credential format and proof mechanism.

A holder uses presentations to respond to verifier requests. A verifier asks for the data it needs. The wallet shows the request to the holder. The holder approves or rejects the disclosure. The wallet then builds a presentation that fits the request and the credential format.

Selective disclosure depends on the credential format and proof mechanism. The W3C VC Data Model defines selective disclosure as the holder's ability to make fine-grained choices about what to share. The same specification treats data minimization as a best practice: verifiers should request the minimum information needed for a transaction. A credential format that supports selective disclosure can let Alice prove she is over 21 without revealing her full birth date. A format that lacks that feature may require sharing more data.

## The Roles in the Trust Triangle

SSI discussions often use the "Triangle of Trust" to describe issuer, holder, and verifier. Treat the triangle as roles in an interaction, not permanent labels for people or organizations.

An issuer creates a credential and takes responsibility for the claims it makes. A department of motor vehicles can issue a driver's license credential. A university can issue a degree credential. A gym can issue a membership credential. The issuer's authority comes from the surrounding context: law, accreditation, contract, community rules, reputation, or governance.

A holder receives credentials and creates presentations from them. The holder may be a person using a mobile wallet, a company using a Cloud Agent, or software managing credentials for devices. Holder control means the holder decides whether to present a credential in a given interaction, subject to wallet design, custody model, law, and policy.

A verifier receives a presentation and decides whether to accept it. Verification usually checks syntax, cryptographic proofs, issuer keys, holder binding, credential status, and schema. Validation checks business rules. For a bar, "over 21 and issued by a recognized state DMV" may be enough. For a hospital employee credential, the verifier may need role, facility, license status, and trust registry checks.

Roles are interaction-specific. The same entity can hold one credential, verify another credential, and issue a third credential.

## Trust Registries and Governance

A DID can prove control over keys. A credential can prove that an issuer made a claim. A verifier still needs a way to decide whether that issuer has the right authority for the verifier's context. Trust registries publish that authorization information for a defined ecosystem.

The Trust over IP Foundation describes a trust registry query in plain terms: "Does entity X hold authorization Y under ecosystem governance framework Z?" The [ToIP Trust Registry Query Protocol v2.0 announcement](https://www.trustoverip.org/blog/2024/04/03/toip-announces-the-implementers-draft-of-thetrust-registry-protocol-specification-v2-0/) frames TRQP as a read-only way to discover who is authorized to do what within a digital trust ecosystem. LF Decentralized Trust describes a trust registry as a system of record containing authoritative information that relying parties use for trust decisions.

A trust registry does not create authority by itself. Authority comes from the governance framework behind it. For a real estate credential, the registry might list licensed agents and the credential types they can issue or verify. For a university degree, a registry might list accredited institutions. For a supply-chain credential, it might list auditors accepted by a particular industry group.

A verifier should treat technical verification and acceptance policy as separate checks. Verification checks proofs, keys, syntax, and status. Acceptance policy checks whether the issuer and credential type fit the relying party's rules.

## Exchange Protocols

DIDs and VCs define data models and verification material. Applications still need protocols to exchange messages, issue credentials, and request presentations.

[DIDComm Messaging v2.1](https://identity.foundation/didcomm-messaging/spec/v2.1/) defines encrypted message formats used by many agent-to-agent SSI systems. DIDComm encrypted messages hide content from everyone except authorized recipients, disclose and prove the sender to those recipients in authenticated mode, and provide integrity guarantees. Identus uses DIDComm V2 for Cloud Agent-to-Cloud Agent communication and for many agent flows.

The OpenID Foundation defines OAuth-based protocols for credential exchange. [OpenID4VCI 1.0](https://openid.net/specs/openid-4-verifiable-credential-issuance-1_0.html) defines an API for issuing Verifiable Credentials. [OpenID4VP 1.0](https://openid.net/specs/openid-4-verifiable-presentations-1_0.html) defines a protocol for requesting and presenting credentials. Production ecosystems may support DIDComm, OpenID4VCI, OpenID4VP, QR-code handoff, offline presentation, or a mix of these paths.

## How This Maps to Identus

Identus gives developers components for these SSI roles, so application code can use agents, SDKs, and APIs instead of reimplementing the standards stack.

The [Identus Cloud Agent](https://hyperledger-identus.github.io/docs/home/quick-start/) can issue, hold, and verify VCs; manage DIDs and DID-based connections; expose a REST API; and use DIDComm V2 for agent communication. Typical uses include custodial wallets, enterprise issuers, verifier services, and backend workflows.

The [Identus DID management documentation](https://hyperledger-identus.github.io/docs/home/identus/cloud-agent/did-management/) explains that Cloud Agent manages PRISM DIDs and Peer DIDs. PRISM DIDs fit public issuer identity and ledger-backed resolution. Peer DIDs fit DIDComm activities where relationship-specific identifiers reduce correlation.

The [Identus Quick Start Guide](https://hyperledger-identus.github.io/docs/home/quick-start/) describes PRISM Node as the Verifiable Data Registry component used to anchor key information for issuance and verification, and describes mediators as services that store and relay messages between Cloud Agents and wallet SDKs. PRISM Node supports public DID resolution. Mediators handle message relay for offline or intermittently connected agents without reading encrypted DIDComm content.

## Developer Takeaways

SSI lets issuers, holders, and verifiers exchange claims with cryptographic proof and holder participation. DIDs identify and resolve verification material. DID Documents publish public keys and services, not private identity profiles. Verifiable Credentials carry issuer claims. Verifiable Presentations let holders respond to verifier requests. Verification checks proofs and status. Validation applies policy. Trust registries and governance help verifiers decide which issuers and credential types count in a specific ecosystem.

In Identus, these ideas map to components: PRISM Node for public DID resolution, Cloud Agent for server-side issuer/holder/verifier work, Edge Agent SDKs for wallets, DIDComm for secure agent messages, and mediators for routing when edge agents are offline.
