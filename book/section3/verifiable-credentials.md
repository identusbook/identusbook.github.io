# Verifiable Credentials {#sec-verifiable-credentials}

## Overview

Think of a Verifiable Credential (VC) as a signed statement that can move with the person or organization it describes. An airline can issue a boarding pass to a traveler. A university can issue a degree credential to a graduate. A government agency can issue a permit to a company. In each case, the holder keeps the credential and later presents it to a verifier that needs proof of a claim.

The [W3C Verifiable Credentials Data Model 2.0](https://www.w3.org/TR/vc-data-model-2.0/) defines the main roles as issuer, holder, verifier, and subject. The issuer creates the credential. The holder stores it and presents it. The verifier checks the presentation. The subject is the person, organization, thing, or account that the claims describe. The holder and subject sometimes match. In other cases, a parent may hold a child credential, or a company administrator may hold credentials about the company.

A VC contains four kinds of information:

- Claims, such as name, ticket number, membership level, age range, or license category.
- Issuer information, so a verifier can decide who made the statement.
- Proof material, so a verifier can check integrity and authorship.
- Metadata, such as issuance date, expiration date, schema, credential type, and status.

The proof lets a verifier detect tampering. It does not decide whether the verifier should accept the credential for a given business purpose. That decision depends on policy. A boarding gate may accept only credentials issued by the airline for the current flight. A verifier may reject an otherwise valid credential if the issuer is not trusted, the credential has expired, the schema is not accepted, or the credential has been revoked.

The [Identus Cloud Agent README](https://github.com/hyperledger-identus/cloud-agent/blob/main/README.md) describes Cloud Agent as a W3C and Aries-based agent that can issue, hold, and verify credentials over DIDComm v2. Identus supports multiple credential formats, exposed through a REST API and webhook-driven agent workflows.

## Formats

VCs share a common purpose, but they do not all use the same envelope, proof format, privacy model, or verification procedure. Identus Cloud Agent documents three credential formats for issuance through its APIs: JWT-VC, SD-JWT-VC, and AnonCreds. In Cloud Agent payloads these appear as `credentialFormat` values such as `JWT`, `SDJWT`, and `AnonCreds`.

JWT-VC packages credential claims in a JSON Web Token. Teams often choose it when their verifiers already rely on JOSE tooling and DID-based issuer verification. The Identus DIDComm issuance guide uses `jwtVcPropertiesV1` for JWT credential offers and notes that this payload shape is aligned with the W3C VC Data Model 1.1 profile used by the agent.

SD-JWT-VC builds on Selective Disclosure JWT. It lets a holder disclose only selected claims from a credential instead of revealing the whole claim set. A traveler proving lounge access may need to disclose membership tier and expiration date, but not date of birth or full customer profile. The [IETF SD-JWT VC draft](https://datatracker.ietf.org/doc/draft-ietf-oauth-sd-jwt-vc/) defines data formats and processing rules for JSON credentials with selective disclosure. The Identus DIDComm issuance guide uses `sdJwtVcPropertiesV1` for this format and requires Ed25519 issuer and holder keys for SD-JWT issuance.

AnonCreds is a zero-knowledge proof credential format. It comes from the Hyperledger Indy ecosystem and is now maintained under [Hyperledger AnonCreds](https://anoncreds.github.io/anoncreds-spec/). Holders use it for privacy-preserving proofs, such as proving predicates over attributes without disclosing raw values. The Identus DIDComm issuance guide uses `anoncredsVcPropertiesV1` for AnonCreds offers. Its claim object is a flat map of string values, and the issuer must have an AnonCreds credential definition.

Early VC 1.1 deployments no longer describe the full standards picture. [W3C announced the Verifiable Credentials 2.0 family as Recommendations on 15 May 2025](https://www.w3.org/news/2025/the-verifiable-credentials-2-0-family-of-specifications-is-now-a-w3c-recommendation/). The [VC JOSE and COSE Recommendation](https://www.w3.org/TR/vc-jose-cose/) defines ways to secure VC Data Model 2.0 credentials and presentations with JOSE, SD-JWT, and COSE. Identus documentation still names the concrete formats and payloads supported by Cloud Agent, so implementation work should follow the Cloud Agent docs first and treat the newer W3C documents as the direction of the wider standards family.

## Schemas

A credential schema describes the shape of credential claims. It answers questions such as: Which fields are expected? Which fields are required? What data type should each field use? A schema does not decide whether a claim is true. It gives issuers, holders, and verifiers a shared structure for reading the credential.

For example, an airline ticket credential might use a schema like this:

```json
{
  "$id": "https://example.com/schemas/boarding-pass-1.0",
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "title": "Boarding pass",
  "type": "object",
  "properties": {
    "passengerName": { "type": "string" },
    "flightNumber": { "type": "string" },
    "departureDate": { "type": "string", "format": "date" },
    "seat": { "type": "string" },
    "boardingGroup": { "type": "string" }
  },
  "required": ["passengerName", "flightNumber", "departureDate"]
}
```

Identus Cloud Agent has a schema registry for creating and resolving credential schemas. The official schema guide documents two creation endpoints:

- `POST /cloud-agent/schema-registry/schemas`, for schemas resolved later by HTTP URL.
- `POST /cloud-agent/schema-registry/schemas/did-url`, for schemas resolved later by DID URL.

Identus supports JSON Schema draft 2020-12 for these schema records. The schema body must set `$schema` to `https://json-schema.org/draft/2020-12/schema`, which matches the current JSON Schema draft described by the [JSON Schema project](https://json-schema.org/learn/getting-started-step-by-step).

The `author` field must be the short-form PRISM DID created by the same Cloud Agent. The DID does not need to be published before schema creation. During setup, an issuer can create a DID, create a schema, and publish the DID later when the credential flow needs public resolution.

Schemas are versioned records. Cloud Agent treats the combination of `author`, schema `id`, and `version` as unique. Updating a schema means creating a new schema record with the changed JSON Schema and a higher version. Do not mutate the meaning of a schema version after credentials have been issued against it. Old credentials remain easier to verify when their original schema still resolves to the same structure.

## Issuing

Issuance is the process that turns an issuer's claim data into a credential held by a wallet or agent. In Identus, issuance can happen through a DIDComm connection, through a connectionless invitation, or through an OpenID for Verifiable Credential Issuance flow.

The Identus DID guides matter before issuance starts. The Cloud Agent registrar can create a PRISM DID offline. The DID document template supports verification methods for purposes such as `authentication` and `assertionMethod`. A long-form PRISM DID carries the create operation in the DID itself. A short-form PRISM DID needs publication before other parties can resolve it from the ledger. The Cloud Agent DID publication guide documents the publication flow and its status changes from `CREATED` to `PUBLICATION_PENDING` to `PUBLISHED`.

For DIDComm issuance, the [Identus Issue Credential Protocol 3.0 guide](https://github.com/hyperledger-identus/cloud-agent/blob/main/docs/docusaurus/credentials/didcomm/issue.md) describes these current prerequisites:

- JWT issuance requires a connection between issuer and holder agents, a published issuer PRISM DID with `assertionMethod`, a credential schema, and a holder PRISM DID with `authentication`.
- SD-JWT issuance has the same shape, but the issuer and holder PRISM DIDs must use Ed25519 keys for the relevant verification methods.
- AnonCreds issuance requires a connection and an AnonCreds credential definition.

From the issuer side, the DIDComm flow starts by creating a credential offer:

```http
POST /issue-credentials/credential-offers
```

The holder retrieves issue credential records and accepts the offer:

```http
GET /issue-credentials/records
PATCH /issue-credentials/records/{recordId}/accept-offer
```

After the holder accepts, the issuer sends the credential:

```http
PATCH /issue-credentials/records/{recordId}/issue-credential
```

The protocol messages are carried over [DIDComm Messaging v2](https://identity.foundation/didcomm-messaging/spec/v2.0/), which is a DID-based secure messaging protocol. In practical terms, the API calls create and advance issue credential records, and the agents exchange the DIDComm messages needed to move the protocol forward.

### Issuer flow

An issuer starts by preparing four pieces of data: the issuer DID, the credential schema or credential definition, the claims, and the credential format.

For a boarding pass, the airline selects its issuer DID, chooses the boarding pass schema, and prepares claims such as passenger name, flight number, departure date, seat, and boarding group. The format choice depends on verifier needs. A gate scanner may only need a JWT-VC. Use SD-JWT-VC when the traveler should reveal only part of the credential. Use AnonCreds when the verifier needs zero-knowledge proofs or predicate proofs.

After creating the offer, the issuer watches the issue credential record state. Identus documents issuer states such as `OfferPending`, `OfferSent`, `RequestReceived`, `CredentialPending`, `CredentialGenerated`, and `CredentialSent`. When the holder sends a request, the issuer can issue the credential from the matching record.

### Holder flow

The holder receives a credential offer, reviews it, and accepts it. In a user-facing wallet, that review step should show the issuer, credential type, requested format, claim preview, and any privacy-relevant details. The holder should know what they are accepting before the wallet stores it.

After acceptance, the holder receives the issued credential over DIDComm. Identus documents holder states such as `OfferReceived`, `RequestPending`, `RequestSent`, and `CredentialReceived`. The wallet stores the credential and later uses it in a presentation flow.

### Connectionless issuance

Connectionless issuance helps when the issuer and holder do not already have a DIDComm connection. The [Identus connectionless issuance guide](https://github.com/hyperledger-identus/cloud-agent/blob/main/docs/docusaurus/credentials/connectionless/issue.md) documents an out-of-band invitation endpoint:

```http
POST /issue-credentials/credential-offers/invitation
```

The issuer creates an invitation and shares the returned invitation code, for example as a QR code or link. The holder accepts the invitation through a wallet or SDK. The issuer responds with a credential offer, and the normal holder acceptance and issuer issuance steps follow.

A conference check-in desk could display a QR code that starts issuance of an attendee credential. The attendee scans it with a wallet, accepts the offer, and receives the credential without first creating a standing relationship with the issuer.

### OpenID4VCI issuance

OpenID for Verifiable Credential Issuance (OpenID4VCI) is another issuance path. The [OpenID Foundation approved OpenID4VCI 1.0 as a Final Specification on 16 September 2025](https://openid.net/openid-for-verifiable-credential-issuance-1-final-specification-approved/). The [specification](https://openid.net/specs/openid-4-verifiable-credential-issuance-1_0.html) defines an API for issuing verifiable credentials where the wallet acts as an OAuth 2.0 client and obtains an access token for the Credential Endpoint.

The [Identus OID4VCI guide](https://github.com/hyperledger-identus/cloud-agent/blob/main/docs/docusaurus/credentials/oid4vci/issue.md) describes Cloud Agent acting as a Credential Issuer server. In that model, Cloud Agent integrates with an Authorization Server that follows the expected contract. Identus provides an example flow with Keycloak and authorization code issuance.

OID4VCI fits issuers that already use OAuth-based user journeys. DIDComm issuance fits wallets and issuers that already participate in agent-to-agent protocols. A production deployment may support both paths, with policy deciding which one applies to each credential type.

## Presenting and verification

Issuance gives the holder a credential. Presentation is how the holder proves something with it.

In Identus, the [Present Proof Protocol 3.0 guide](https://github.com/hyperledger-identus/cloud-agent/blob/main/docs/docusaurus/credentials/didcomm/present-proof.md) describes verifier, holder, and prover flows over DIDComm. A verifier creates a proof request:

```http
POST /present-proof/presentations
```

The request can include a `domain` and `challenge`. These values help bind the presentation to the verifier and session, which reduces replay risk. The holder reviews the request, selects credentials, and accepts it:

```http
PATCH /present-proof/presentations/{id}
```

The exact payload depends on the credential format. JWT presentations use a `proofId`. SD-JWT presentations can disclose selected claims and use `credentialFormat: SDJWT`. AnonCreds presentations use an AnonCreds presentation request. Identus documents a special SD-JWT case: if the credential has no `cnf` key, the holder cannot create and sign a presentation bound to the challenge and domain. If it has a `cnf` key, the holder can create that signed presentation.

After verification, Identus can reach `PresentationVerified`. The verifier can then accept the presentation, which moves the record to `PresentationAccepted`. This split matters. Verification checks cryptographic validity and protocol rules. Acceptance records a business decision by the verifier.

## Revoking

Revocation lets an issuer mark a credential as no longer valid before its natural expiration. An airline may revoke a boarding pass after a booking is canceled. A university may revoke a credential issued by mistake. A company may revoke an employee credential after employment ends.

The [Identus revocation guide](https://github.com/hyperledger-identus/cloud-agent/blob/main/docs/docusaurus/credentials/revocation.md) documents revocation for JWT credentials using Verifiable Credentials Status List v2021. A revocable credential contains a `credentialStatus` object with fields such as `type`, `statusPurpose`, `statusListIndex`, and `statusListCredential`.

The status list credential contains an encoded bitstring. During verification, the verifier resolves the `statusListCredential`, verifies its proof, decodes the list, and checks the bit at `statusListIndex`. If the bit is set, the credential is revoked. If the bit is not set, the credential has not been revoked by that status list.

Only the issuer of a credential can revoke it. Identus exposes the revocation operation as:

```http
PATCH /cloud-agent/revoke-credential/{credential_id}
```

Once a credential is revoked, a later Present Proof verification fails if the holder presents that credential. This keeps revocation in the verification path instead of relying on the holder to remove old credentials from storage.

W3C published [Bitstring Status List v1.0](https://www.w3.org/TR/vc-bitstring-status-list/) as part of the VC 2.0 Recommendation family on 15 May 2025. The Identus documentation cited above still names Status List v2021 for JWT credential revocation, so implementers should follow the current Identus guide for Cloud Agent behavior and track future Identus releases for any status-list migration.
