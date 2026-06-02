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

The [Identus Cloud Agent README](https://github.com/hyperledger-identus/cloud-agent/blob/main/README.md) describes Cloud Agent as a W3C and Aries-based agent that can issue, hold, and verify credentials over DIDComm v2. Identus supports multiple credential formats through a REST API and webhook-driven agent workflows.

## Formats

VCs share a common purpose, but they do not all use the same envelope, proof format, privacy model, or verification procedure. Identus Cloud Agent documents three credential formats for issuance through its APIs: JWT-VC, SD-JWT-VC, and AnonCreds. In Cloud Agent payloads these appear as `credentialFormat` values such as `JWT`, `SDJWT`, and `AnonCreds`.

### JWT Verifiable Credentials

JWT-VC packages credential claims in a JSON Web Token. Teams often choose it when their verifiers already rely on JOSE tooling and DID-based issuer verification. The verifier decodes the JWT, validates the signature against the issuer's public key resolved from the issuer DID, and reads the claims.

JWT-VC has a privacy tradeoff. The holder usually presents the credential as a whole, so the verifier sees all claims in the credential. That is fine for a boarding gate that needs the full ticket details. It is a poor fit when the holder should reveal only one or two fields.

The Identus DIDComm issuance guide uses `jwtVcPropertiesV1` for JWT credential offers and notes that this payload shape is aligned with the W3C VC Data Model 1.1 profile used by the agent.

### SD-JWT Verifiable Credentials

SD-JWT-VC builds on Selective Disclosure JWT. It lets a holder disclose selected claims from a credential instead of revealing the whole claim set. A traveler proving lounge access may need to disclose membership tier and expiration date, but not date of birth or full customer profile.

The [IETF SD-JWT VC draft](https://datatracker.ietf.org/doc/draft-ietf-oauth-sd-jwt-vc/) defines data formats and processing rules for JSON credentials with selective disclosure. At a high level, the issuer signs a structure that commits to the claim values. The holder later presents only the disclosures needed for the verifier's request, and the verifier checks those disclosures against the signed commitments.

Identus uses `sdJwtVcPropertiesV1` for this format and requires Ed25519 issuer and holder keys for SD-JWT issuance. Identus Present Proof documentation calls out one presentation detail: if the SD-JWT credential has no `cnf` key, the holder cannot create and sign a presentation bound to the verifier's challenge and domain. With `cnf`, the holder can prove control of the bound key during presentation.

### AnonCreds

AnonCreds is a zero-knowledge proof credential format. It comes from the Hyperledger Indy ecosystem and is now maintained under [Hyperledger AnonCreds](https://anoncreds.github.io/anoncreds-spec/). Holders use it for privacy-preserving proofs, such as proving predicates over attributes without disclosing raw values.

AnonCreds requires a credential definition. That artifact references a schema and contains issuer cryptographic material used for proof generation. In Identus, AnonCreds claims use a flat string-to-string shape, and the issuer references the credential definition with `credentialDefinitionId` inside `anoncredsVcPropertiesV1`.

### OID4VCI

OpenID for Verifiable Credential Issuance (OpenID4VCI) is an issuance protocol, not a credential format. It extends OAuth 2.0-style flows so a wallet can obtain credentials from an issuer through a Credential Endpoint. The [OpenID Foundation approved OpenID4VCI 1.0 as a Final Specification on 16 September 2025](https://openid.net/openid-for-verifiable-credential-issuance-1-final-specification-approved/), and the [specification](https://openid.net/specs/openid-4-verifiable-credential-issuance-1_0.html) defines the issuer metadata, credential configurations, authorization flow, and credential request flow.

The [Identus OID4VCI guide](https://github.com/hyperledger-identus/cloud-agent/blob/main/docs/docusaurus/credentials/oid4vci/issue.md) describes Cloud Agent acting as a Credential Issuer server and integrating with an Authorization Server that follows the expected contract. Identus provides an example flow with Keycloak and authorization code issuance.

### W3C VC 2.0 context

Early VC 1.1 deployments no longer describe the full standards picture. [W3C announced the Verifiable Credentials 2.0 family as Recommendations on 15 May 2025](https://www.w3.org/news/2025/the-verifiable-credentials-2-0-family-of-specifications-is-now-a-w3c-recommendation/). The [VC JOSE and COSE Recommendation](https://www.w3.org/TR/vc-jose-cose/) defines ways to secure VC Data Model 2.0 credentials and presentations with JOSE, SD-JWT, and COSE.

Identus documentation still names the concrete formats and payloads supported by Cloud Agent, so implementation work should follow the Cloud Agent docs first and treat the W3C VC 2.0 documents as the direction of the wider standards family.

## Schemas

A credential schema describes the shape of credential claims. It answers questions such as: Which fields are expected? Which fields are required? What data type should each field use? A schema does not decide whether a claim is true. It gives issuers, holders, and verifiers a shared structure for reading the credential.

### Schema specifications

Identus uses two schema families across its credential formats.

JWT and SD-JWT credentials use JSON Schema. The official Identus schema guide requires the schema body to set `$schema` to `https://json-schema.org/draft/2020-12/schema`, which matches the current draft described by the [JSON Schema project](https://json-schema.org/learn/getting-started-step-by-step).

AnonCreds credentials use AnonCreds schemas paired with credential definitions. The credential definition references a schema by ID and must exist before the issuer can create an AnonCreds credential offer.

### Example schema

An airline ticket credential might use a JSON Schema like this:

```json
{
  "$id": "https://example.com/schemas/boarding-pass-1.0",
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "title": "Boarding pass",
  "type": "object",
  "properties": {
    "passengerName": { "type": "string" },
    "flightNumber": { "type": "string" },
    "departureAirport": { "type": "string" },
    "arrivalAirport": { "type": "string" },
    "departureDateTime": { "type": "string", "format": "date-time" },
    "seatNumber": { "type": "string" },
    "boardingGroup": { "type": "string" },
    "ticketClass": { "type": "string" }
  },
  "required": [
    "passengerName",
    "flightNumber",
    "departureAirport",
    "arrivalAirport",
    "departureDateTime",
    "seatNumber"
  ],
  "additionalProperties": false
}
```

### Creating schemas in Identus

Identus Cloud Agent has a schema registry for creating and resolving credential schemas. The official schema guide documents two creation endpoints:

```http
POST /cloud-agent/schema-registry/schemas
POST /cloud-agent/schema-registry/schemas/did-url
```

Use the HTTP endpoint when verifiers should resolve the schema through an HTTP URL. Use the DID URL endpoint when verifiers should resolve the schema through a DID URL. These are separate resolution modes, so keep the expected verifier path in mind before issuing credentials against the schema.

Each schema record includes metadata around the schema definition. The `name` gives the schema a readable label, `version` identifies the version, `author` names the DID that created it, and `tags` help filter schema records. The `schema` field contains the JSON Schema definition. Cloud Agent treats the combination of `author`, schema `id`, and `version` as unique.

The `author` field must be the short-form PRISM DID created by the same Cloud Agent. The DID does not need to be published before schema creation. During setup, an issuer can create a DID, create a schema, and publish the DID later when the credential flow needs public resolution.

### Versioning

Updating a schema means creating a new schema record with the changed JSON Schema and a higher version. Do not mutate the meaning of a schema version after credentials have been issued against it. Old credentials remain easier to verify when their original schema still resolves to the same structure.

For breaking changes, such as removing required fields or changing field types, use a new major version. For additive changes, such as adding an optional field, use a minor version. This keeps issuer and verifier policy readable when multiple credential versions exist at the same time.

## Issuing

Issuance turns an issuer's claim data into a credential held by a wallet or agent. In Identus, issuance can happen through a DIDComm connection, through a connectionless invitation, or through OpenID4VCI.

The Identus DID guides matter before issuance starts. The Cloud Agent registrar can create a PRISM DID offline. The DID document template supports verification methods for purposes such as `authentication` and `assertionMethod`. A long-form PRISM DID carries the create operation in the DID itself. A short-form PRISM DID needs publication before other parties can resolve it from the ledger. The Cloud Agent DID publication guide documents the publication flow and its status changes from `CREATED` to `PUBLICATION_PENDING` to `PUBLISHED`.

### Prerequisites

For DIDComm issuance, the [Identus Issue Credential Protocol 3.0 guide](https://github.com/hyperledger-identus/cloud-agent/blob/main/docs/docusaurus/credentials/didcomm/issue.md) describes the format-specific prerequisites.

JWT issuance requires a connection between issuer and holder agents, a published issuer PRISM DID with `assertionMethod`, a credential schema, and a holder PRISM DID with `authentication`.

SD-JWT issuance has the same shape, but the issuer and holder PRISM DIDs must use Ed25519 keys for the relevant verification methods.

AnonCreds issuance requires a connection and an AnonCreds credential definition.

### Credential states

The issuer-side states are:

```text
OfferPending -> OfferSent -> RequestReceived -> CredentialPending -> CredentialGenerated -> CredentialSent
```

The holder-side states are:

```text
OfferReceived -> RequestPending -> RequestSent -> CredentialReceived
```

These states are useful when a wallet or controller needs to resume a flow, show progress to a user, or decide whether a manual issuer action is still required.

### Issuer flow

An issuer starts by preparing four pieces of data: the issuer DID, the credential schema or credential definition, the claims, and the credential format.

For a boarding pass, the airline selects its issuer DID, chooses the boarding pass schema, and prepares claims such as passenger name, flight number, departure date, seat, and boarding group. The format choice depends on verifier needs. A gate scanner may only need a JWT-VC. Use SD-JWT-VC when the traveler should reveal only part of the credential. Use AnonCreds when the verifier needs zero-knowledge proofs or predicate proofs.

The issuer creates a credential offer through Cloud Agent:

```http
POST /cloud-agent/issue-credentials/credential-offers
```

JWT offers use `jwtVcPropertiesV1`:

```json
{
  "connectionId": "holder-connection-id",
  "credentialFormat": "JWT",
  "jwtVcPropertiesV1": {
    "claims": {
      "passengerName": "Alice Wonderland",
      "flightNumber": "ID123",
      "departureAirport": "SCL",
      "arrivalAirport": "EZE",
      "seatNumber": "12A"
    },
    "issuingDID": "did:prism:issuer",
    "credentialSchema": {
      "id": "http://localhost:8080/cloud-agent/schema-registry/schemas/...",
      "type": "JsonSchemaValidator2018"
    }
  }
}
```

SD-JWT offers use `sdJwtVcPropertiesV1`:

```json
{
  "connectionId": "holder-connection-id",
  "credentialFormat": "SDJWT",
  "sdJwtVcPropertiesV1": {
    "claims": {
      "passengerName": "Alice Wonderland",
      "flightNumber": "ID123",
      "boardingGroup": "A",
      "exp": 1883000000
    },
    "issuingDID": "did:prism:issuer",
    "credentialSchema": {
      "id": "http://localhost:8080/cloud-agent/schema-registry/schemas/...",
      "type": "JsonSchemaValidator2018"
    }
  }
}
```

AnonCreds offers use `anoncredsVcPropertiesV1` and a `credentialDefinitionId`:

```json
{
  "connectionId": "holder-connection-id",
  "credentialFormat": "AnonCreds",
  "anoncredsVcPropertiesV1": {
    "claims": {
      "passengerName": "Alice Wonderland",
      "flightNumber": "ID123",
      "seatNumber": "12A"
    },
    "issuingDID": "did:prism:issuer",
    "credentialDefinitionId": "credential-definition-id"
  }
}
```

After creating the offer, the issuer watches the issue credential record state. If the holder accepts the offer and the issuer uses automatic issuance, Cloud Agent issues the credential when the holder's request arrives. If automatic issuance is disabled, the issuer sends the credential from the matching record:

```http
POST /cloud-agent/issue-credentials/records/{recordId}/issue-credential
```

### Holder flow

The holder receives a credential offer, reviews it, and accepts it. In a user-facing wallet, that review step should show the issuer, credential type, requested format, claim preview, and any privacy-relevant details. The holder should know what they are accepting before the wallet stores it.

The holder can retrieve issue credential records:

```http
GET /cloud-agent/issue-credentials/records
```

To accept a JWT or SD-JWT offer, the holder provides a PRISM DID as `subjectId`:

```json
{
  "subjectId": "did:prism:holder"
}
```

For SD-JWT credentials with key binding, the holder can include `keyId`:

```json
{
  "subjectId": "did:prism:holder",
  "keyId": "key-1"
}
```

AnonCreds offers do not use `subjectId` in the same way. The holder accepts the offer with an empty body:

```json
{}
```

After acceptance, the holder receives the issued credential over DIDComm. Identus documents holder states such as `OfferReceived`, `RequestPending`, `RequestSent`, and `CredentialReceived`. The wallet stores the credential and later uses it in a presentation flow.

### Connectionless issuance

Connectionless issuance helps when the issuer and holder do not already have a DIDComm connection. The [Identus connectionless issuance guide](https://github.com/hyperledger-identus/cloud-agent/blob/main/docs/docusaurus/credentials/connectionless/issue.md) documents an out-of-band invitation endpoint:

```http
POST /cloud-agent/issue-credentials/credential-offers/invitation
```

The issuer creates an invitation and shares the returned invitation code, for example as a QR code or link. The holder accepts the invitation through a wallet or SDK. The issuer responds with a credential offer, and the normal holder acceptance and issuer issuance steps follow.

A conference check-in desk could display a QR code that starts issuance of an attendee credential. The attendee scans it with a wallet, accepts the offer, and receives the credential without first creating a standing relationship with the issuer.

### OpenID4VCI issuance

OID4VCI fits issuers that already use OAuth-based user journeys. DIDComm issuance fits wallets and issuers that already participate in agent-to-agent protocols. A production deployment may support both paths, with policy deciding which one applies to each credential type.

In the Identus OID4VCI model, Cloud Agent acts as the Credential Issuer server. The wallet follows the authorization flow, obtains an access token from the Authorization Server, and uses that token at the Credential Endpoint. The exact issuer policy lives in the issuer and authorization server configuration, not in the wallet.

## Presenting and verification

Issuance gives the holder a credential. Presentation is how the holder proves something with it.

In Identus, the [Present Proof Protocol 3.0 guide](https://github.com/hyperledger-identus/cloud-agent/blob/main/docs/docusaurus/credentials/didcomm/present-proof.md) describes verifier, holder, and prover flows over DIDComm. A verifier creates a proof request:

```http
POST /cloud-agent/present-proof/presentations
```

The request can include a `domain` and `challenge`. These values bind the presentation to the verifier and session, which reduces replay risk. The holder reviews the request, selects credentials, and accepts it:

```http
PATCH /cloud-agent/present-proof/presentations/{id}
```

The exact payload depends on the credential format. JWT presentations use a `proofId`. SD-JWT presentations disclose selected claims and use `credentialFormat: SDJWT`. AnonCreds presentations use an AnonCreds presentation request.

After verification, Identus can reach `PresentationVerified`. The verifier can then accept the presentation, which moves the record to `PresentationAccepted`. This split matters. Verification checks cryptographic validity and protocol rules. Acceptance records a business decision by the verifier.

In the airline example, a valid ticket credential may still fail business policy. A traveler can hold a legitimate ticket and still lack a required visa, arrive after the gate closes, or fail a security check. VC verification tells the verifier whether the credential is authentic, intact, and acceptable under protocol rules. The verifier still decides whether to accept it for the trip.

## Revoking

Revocation lets an issuer mark a credential as no longer valid before its natural expiration. An airline may revoke a boarding pass after a booking is canceled. A university may revoke a credential issued by mistake. A company may revoke an employee credential after employment ends.

The [Identus revocation guide](https://github.com/hyperledger-identus/cloud-agent/blob/main/docs/docusaurus/credentials/revocation.md) documents revocation for JWT credentials using Verifiable Credentials Status List v2021. A revocable credential contains a `credentialStatus` object with fields such as `type`, `statusPurpose`, `statusListIndex`, and `statusListCredential`.

### Revocation mechanism

A revocable credential contains status data like this:

```json
{
  "credentialStatus": {
    "id": "http://localhost:8080/cloud-agent/credential-status/[UUID]#[INDEX]",
    "type": "StatusList2021Entry",
    "statusPurpose": "revocation",
    "statusListIndex": "94567",
    "statusListCredential": "http://localhost:8080/cloud-agent/credential-status/[UUID]"
  }
}
```

The `statusListCredential` URL points to a Status List Credential. That credential contains an encoded bitstring where each bit position corresponds to a credential. The `statusListIndex` identifies the credential's position in that bitstring. When the issuer revokes a credential, the issuer flips the corresponding bit from 0 to 1.

### Revoking a credential

Only the issuer of a credential can revoke it. Identus exposes the revocation operation as:

```http
PATCH /cloud-agent/revoke-credential/{credential_id}
```

The issuer can find the credential ID by listing issued credential records, then calling the revocation endpoint for the target credential. Once revoked, a later Present Proof verification fails if the holder presents that credential. This keeps revocation in the verification path instead of relying on the holder to remove old credentials from storage.

### Checking revocation status

When a verifier receives a presentation containing a credential with `credentialStatus`, the verifier checks the status list before accepting the credential.

First, the verifier retrieves the Status List Credential from `credentialStatus.statusListCredential`. Then it verifies the proof embedded in that status credential. Identus documents two supported proof types for status list credentials: `DataIntegrityProof` with `eddsa-jcs-2022` for Ed25519 keys, and `EcdsaSecp256k1Signature2019` for secp256k1 keys.

After proof verification, the verifier decodes `credentialSubject.encodedList` and checks the bit at `statusListIndex`. If the bit is set to 1, the credential has been revoked. If the bit is 0, the status list has not revoked that credential.

This status-list model has a privacy benefit: the verifier retrieves the whole list instead of asking the issuer about a specific credential. The issuer does not learn which credential the verifier checked.

W3C published [Bitstring Status List v1.0](https://www.w3.org/TR/vc-bitstring-status-list/) as part of the VC 2.0 Recommendation family on 15 May 2025. The Identus documentation cited above still names Status List v2021 for JWT credential revocation, so implementers should follow the current Identus guide for Cloud Agent behavior and track future Identus releases for any status-list migration.
