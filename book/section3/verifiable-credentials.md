# Verifiable Credentials {#sec-verifiable-credentials}

## Overview

Verifiable Credentials (VCs) are a central element of Self-Sovereign Identity (SSI), giving individuals the ability to manage and control the sharing of their personal information. Throughout this book, we will explore VCs by building an example airline ticket wallet, where a Verifiable Credential serves as a digital flight ticket, illustrating the concepts in a practical manner.

A Verifiable Credential is a digital assertion made by an **Issuer** concerning a **Subject**. This assertion is cryptographically secured, allowing a **Verifier** to confirm its authenticity and integrity without needing to directly contact the Issuer. VCs can represent many kinds of information, from identity documents and academic degrees to professional certifications and, as in our project, airline tickets. They provide a digital, secure, and verifiable alternative to traditional paper-based credentials.

A VC has several key components. The **Issuer** is the entity that creates and cryptographically signs the credential; in our airline example, this is the Airline company responsible for issuing flight tickets. The **Holder** is the individual or entity to whom the credential is issued and who controls its presentation; this role is filled by the passenger who has purchased the ticket. When the passenger needs to prove their right to board a flight, they present their ticket to a **Verifier**, in this case, the Airport Security Officer, who then checks the credential's authenticity and validity. The **Subject** is the entity the credential's claims are about. Very often, as with our example, the Holder and the Subject are one and the same; in our case, the passenger. The credential contains **Claims**, which are specific statements about the Subject, such as the passenger's name, flight number, and seat assignment for an airline ticket. The security and trustworthiness of a VC are guaranteed by its **Proof**, typically a digital signature, which provides cryptographic evidence of authenticity and integrity. **Metadata** offers additional context, such as the credential's expiration date or a description of its purpose, like "Airline Boarding Pass."

The lifecycle of a Verifiable Credential involves several key stages, which we will implement in our example project:

The **Issuance** process is how a VC is created and delivered. As per the Identus Cloud Agent's implementation of the Issue Credential Protocol 3.0, this flow is initiated by the Issuer. For instance, the Airline (Issuer) would first create a credential offer, a proposal to issue a specific ticket, and send it to the passenger (Holder) via DIDComm. If the passenger accepts this offer, they respond with a credential request. The Airline then issues the Verifiable Credential containing claims like flight details and the passenger's DID. In the last step of issuance, the Issuer signs with their private key and sends it to the Holder.

Once received, the **Storage** phase begins. The passenger (Holder) stores this digital ticket securely in their digital wallet, such as the airline ticket wallet mobile application we will develop.

When the passenger passes through security at the airport, a **Presentation** occurs. The passenger (Holder) presents their digital airline ticket to the Airport Security Officer (Verifier). This can involve presenting the entire credential or, through mechanisms like Selective Disclosure, only specific relevant claims (e.g., just the name and flight number, without revealing the ticket price or other details), preserving the Holder's privacy.

**Verification** is performed by the Airport Security Officer. They use their systems to check the digital signature on the ticket to confirm its authenticity (i.e., it was indeed issued by the legitimate Airline) and integrity (i.e., it has not been tampered with since issuance). This step confirms that the ticket is valid and trustworthy.

A note on **Trust**. Just because a Verifiable Credential is legitimate and passes the cryptographic verification, it doesn't mean the presentation will be accepted. There might be other business rules and checks needed to establish a final decision. For example, the passenger might be part of a "no fly" list or lack a required visa for international travel, and even though they hold a legitimate ticket, the Airport Security Officer might reject the presentation. The important insight is that after the presentation reaches the verified status, there needs to be a **manual** final decision to accept or reject a presentation.

Verifiable Credentials offer significant advantages. Their adherence to standard formats promotes **Interoperability**, meaning they can be used across diverse ecosystems and platforms without vendor lock-in. **Privacy** is a key benefit, since Holders can selectively disclose only the necessary information rather than revealing all data contained within a credential, giving them granular control over their personal data. Cryptographic techniques provide strong **Security**, guaranteeing the integrity and authenticity of credentials against tampering and forgery. VCs used within Self-Sovereign Identity interactions support **Decentralization**, as verification often doesn't require real-time communication with the original Issuer or reliance on a central authority, reducing single points of failure and control.

## Formats

Identus supports multiple Verifiable Credential formats, each with distinct characteristics suited for different use cases. The choice of format affects how claims are structured, how selective disclosure works, and what cryptographic mechanisms are used for proofs.

### JWT Verifiable Credentials (W3C v1.1)

The [W3C Verifiable Credentials Data Model v1.1](https://www.w3.org/TR/vc-data-model/) is the most widely adopted standard for VCs. In Identus, this format encodes VCs as JSON Web Tokens (JWTs), where the credential claims are placed in the JWT payload and the issuer's digital signature serves as the proof. JWT VCs are straightforward to implement and verify: the verifier decodes the JWT, validates the signature against the issuer's public key (resolved from their DID), and reads the claims. The trade-off is that JWT VCs require the holder to present all claims at once -- the verifier sees everything in the credential, with no built-in mechanism for hiding individual fields.

Identus fully supports JWT VCs and they are the default credential format. When creating a credential offer, the issuer specifies `"credentialFormat": "JWT"` and provides claims as key-value pairs within the `jwtVcPropertiesV1` object.

### SD-JWT Verifiable Credentials

[SD-JWT (Selective Disclosure JSON Web Token)](https://datatracker.ietf.org/doc/draft-ietf-oauth-sd-jwt-vc/) extends the JWT format with selective disclosure capabilities. Instead of exposing all claims at once, each claim in an SD-JWT VC is individually hashed and salted. The issuer signs the credential containing these hashed values, and the holder receives the original claim values alongside the signed JWT. When presenting the credential, the holder can choose which claims to reveal by including only the corresponding disclosure values. The verifier can then hash the disclosed values and compare them against the signed hashes to confirm they were part of the original credential.

SD-JWT VCs in Identus support optional key binding, where the credential is cryptographically tied to a specific key held by the holder. This prevents credential replay, because the holder must prove possession of the bound key during presentation. To use SD-JWT, the issuer specifies `"credentialFormat": "SDJWT"` and provides claims within the `sdJwtVcPropertiesV1` object. Both the issuer's and holder's PRISM DIDs must use the Ed25519 curve.

### AnonCreds

[AnonCreds (Anonymous Credentials)](https://github.com/hyperledger/anoncreds-spec) is a credential format originating from the Hyperledger ecosystem that provides strong privacy through zero-knowledge proofs. With AnonCreds, a holder can prove that specific claims meet certain conditions without revealing the underlying data. For example, a holder can prove they are over 21 without disclosing their exact date of birth, or prove they hold a valid credential from a trusted issuer without revealing which specific credential they hold, preventing cross-verifier tracking.

AnonCreds requires a **credential definition**, a separate artifact that references a schema and contains the issuer's cryptographic material needed for zero-knowledge proof generation. In Identus, AnonCreds claims are restricted to flat string key-value pairs (no nested objects), and the issuer must use the Ed25519 curve. The credential format is specified as `"credentialFormat": "AnonCreds"` with claims in the `anoncredsVcPropertiesV1` object.

### OID4VCI

[OID4VCI (OpenID for Verifiable Credential Issuance)](https://openid.net/2023/02/22/oid4vci-1-0-release/) is not a credential format itself but an issuance protocol that extends OAuth 2.0 to support credential delivery. It allows holders to obtain VCs from issuers using familiar OAuth flows, including authorization code grants and pre-authorized code flows. Identus supports OID4VCI as an alternative to DIDComm-based issuance, which can be useful in scenarios where the holder interacts with the issuer through a web browser or an application that already uses OAuth.

### W3C v2.0

The [W3C Verifiable Credentials Data Model v2.0](https://www.w3.org/TR/vc-data-model-2.0/) is the next generation of the W3C standard, introducing improvements such as a more flexible data model, better support for multiple proof mechanisms, and tighter integration with the broader W3C ecosystem. This format is on the Identus roadmap for future support.

## Schemas

A credential schema serves as a structural template that defines which claims a Verifiable Credential can contain. When an issuer creates a credential, the schema acts as a contract specifying the expected fields, their data types, and which fields are required. Verifiers can reference the same schema to confirm that a credential conforms to an expected structure before processing its claims.

Schemas can optionally be published on a Verifiable Data Registry (VDR), which is particularly beneficial for widely applicable schema types. Publishing schemas on a VDR facilitates their adoption by other parties, enabling third parties to issue VCs that conform to the same standardized credential format. For example, relevant entities or industry consortiums can collaboratively develop, agree upon, and publish schemas that they will adopt and recognize, encouraging other players in the ecosystem to do the same and fostering interoperability and growth.

### Schema Specifications

Identus supports two credential schema specifications:

The **W3C Verifiable Credentials JSON Schema 2022** is used for JWT and SD-JWT credentials. Schemas in this format follow the JSON Schema standard (specifically `draft/2020-12/schema`) and describe the shape of the credential's claims using standard JSON Schema constructs: `properties` for field definitions, `type` for data types, `required` for mandatory fields, and `format` for semantic annotations like `email` or `date-time`.

The **AnonCreds Schema** follows the Hyperledger AnonCreds specification and is used when issuing AnonCreds credentials. AnonCreds schemas have a simpler structure and are paired with a **credential definition**, a separate artifact containing the issuer's cryptographic material needed for zero-knowledge proof generation. The credential definition references a schema by its ID, and both must exist before the issuer can create AnonCreds credential offers.

### Schema Attributes

Each schema record in Identus carries several attributes beyond the JSON Schema definition itself. The `guid` is a globally unique identifier derived from the combination of the author's DID, the schema ID, and the version. The `name` provides a human-readable label for the schema (e.g., "Airline Boarding Pass"), and the `version` follows semantic versioning (SemVer). The `author` field holds the DID of the entity that created the schema, and `authored` records the creation timestamp in RFC 3339 format. An optional `tags` array can be used for filtering and categorizing schemas. The `schema` field contains the actual JSON Schema definition, and the `proof` field holds the JOSE signature object when the schema is signed.

### Example Schema

Here is an example of a credential schema for a driving license:

```json
{
  "$id": "https://example.com/driving-license-1.0",
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "description": "Driving License",
  "type": "object",
  "properties": {
    "emailAddress": {
      "type": "string",
      "format": "email"
    },
    "givenName": {
      "type": "string"
    },
    "familyName": {
      "type": "string"
    },
    "dateOfIssuance": {
      "type": "string",
      "format": "date-time"
    },
    "drivingLicenseID": {
      "type": "string"
    },
    "drivingClass": {
      "type": "integer"
    }
  },
  "required": [
    "emailAddress",
    "familyName",
    "dateOfIssuance",
    "drivingLicenseID",
    "drivingClass"
  ],
  "additionalProperties": true
}
```

### Creating a Schema

Identus exposes two REST API endpoints for schema creation. The first creates schemas resolvable through a standard HTTP URL, and the second creates schemas resolvable via a DID URL:

```
POST /cloud-agent/schema-registry/schemas
POST /cloud-agent/schema-registry/schemas/did-url
```

The request payload combines metadata about the schema with the JSON Schema definition. Here is an example that creates a schema for our airline boarding pass:

```json
{
  "name": "AirlineBoardingPass",
  "version": "1.0.0",
  "description": "Schema for airline boarding pass credentials",
  "type": "https://w3c-ccg.github.io/vc-json-schemas/schema/2.0/schema.json",
  "author": "did:prism:issuer_did_here",
  "tags": ["airline", "boarding-pass", "travel"],
  "schema": {
    "$id": "https://example.com/airline-boarding-pass-1.0",
    "$schema": "https://json-schema.org/draft/2020-12/schema",
    "description": "Airline Boarding Pass",
    "type": "object",
    "properties": {
      "passengerName": {
        "type": "string"
      },
      "flightNumber": {
        "type": "string"
      },
      "departureAirport": {
        "type": "string"
      },
      "arrivalAirport": {
        "type": "string"
      },
      "departureDateTime": {
        "type": "string",
        "format": "date-time"
      },
      "seatNumber": {
        "type": "string"
      },
      "boardingGroup": {
        "type": "integer"
      },
      "ticketClass": {
        "type": "string"
      }
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
}
```

The `author` field must match the short form of a PRISM DID that was created on the same agent. The `$schema` field within the schema definition must be `"https://json-schema.org/draft/2020-12/schema"`, which is the only supported draft version. The combination of author DID, schema ID, and version must be unique across the agent's schema registry.

Schemas created through the HTTP endpoint return a JSON response containing the schema's `guid`, `self` link, and all metadata. Schemas created through the DID URL endpoint return a Prism Envelope containing a base64-encoded resource and a DID URL for resolution.

### Schema Versioning

Schemas follow semantic versioning rules. When making changes to an existing schema, you create a new schema record with an incremented version number. Breaking changes (removing required fields, changing field types) call for a major version increment. HTTP-based and DID-based schemas are stored separately and are not cross-compatible when retrieving them through their respective endpoints.

### Schema Publishing and Current Limitations

Once a schema has been published to a VDR, it cannot be deleted, which protects the integrity of any credentials that depend on it.

::: {.callout-note}
In the current Identus implementation, the Issuer does not sign the Credential Schema and does not publish it to the VDR (the Cardano blockchain). Schema storage and management are handled internally by the Cloud Agent. VDR publishing is planned for a future release.
:::

## Issuing

Issuing a Verifiable Credential is a multi-step exchange between an issuer and a holder. The process follows the Issue Credential Protocol 3.0, where the issuer initiates the flow by creating a credential offer and the holder responds by accepting the offer and receiving the signed credential. Identus currently supports issuance through the Cloud Agent's REST API endpoints and via the OID4VCI protocol.

### Prerequisites

All three credential formats share a common requirement: an established connection between the issuing Cloud Agent and the holder. The holder can be another Cloud Agent or an edge client device connected through a mediator.

Beyond this shared prerequisite, each format has specific requirements:

For **JWT** credentials, the issuing agent must have a published PRISM DID with an `assertionMethod` key for signing credentials. The holder needs a PRISM DID with an `authentication` key. A credential schema must exist in the agent's schema registry.

For **SD-JWT** credentials, the requirements match those of JWT, with the added constraint that both the issuer's and holder's PRISM DIDs must use the Ed25519 elliptic curve. The holder can optionally enable key binding by specifying a `keyId` when accepting the offer.

For **AnonCreds** credentials, the issuing agent needs a published PRISM DID using the Ed25519 curve. A credential schema and a corresponding credential definition must both exist before the issuer can create an offer. The holder does not need a PRISM DID for AnonCreds.

### Credential States

Credentials in Identus transition through a well-defined set of states during the issuance process. Tracking these states is useful for monitoring the progress of credential exchanges.

On the **issuer** side, the state transitions are: `OfferPending` (offer created, waiting to be sent) -> `OfferSent` (offer delivered to holder via DIDComm) -> `RequestReceived` (holder has accepted and sent back a credential request) -> `CredentialPending` (issuer is preparing the credential) -> `CredentialGenerated` (credential signed and ready) -> `CredentialSent` (credential delivered to the holder).

On the **holder** side, the state transitions are: `OfferReceived` (offer arrived from issuer) -> `RequestPending` (holder accepted, request being prepared) -> `RequestSent` (request sent to issuer) -> `CredentialReceived` (signed credential received and stored).

You can check the state of any credential record at any point by querying the API:

```bash
curl "http://localhost:8080/cloud-agent/issue-credentials/records/{recordId}" \
  -H "Content-Type: application/json" \
  -H "apikey: $API_KEY"
```

### Issuer Flow

The API endpoint for creating a credential offer is:

```
POST /cloud-agent/issue-credentials/credential-offers
```

The structure of the request body varies depending on the credential format. Here is how the Airline (issuer) would create a JWT credential offer for a boarding pass:

```bash
curl -X POST 'http://localhost:8080/cloud-agent/issue-credentials/credential-offers' \
  -H 'Content-Type: application/json' \
  -H "apikey: $API_KEY" \
  -d '{
    "connectionId": "9d075518-f97e-4f11-9d10-d7348a7a0fda",
    "credentialFormat": "JWT",
    "jwtVcPropertiesV1": {
      "claims": {
        "emailAddress": "alice@wonderland.com",
        "givenName": "Alice",
        "familyName": "Wonderland"
      },
      "issuingDID": "did:prism:9f847f8bbb66c112f71d08ab39930d468ccbfe1e0e1d002be53d46c431212c26",
      "credentialSchema": {
        "id": "http://localhost:8080/cloud-agent/schema-registry/schemas/...",
        "type": "JsonSchemaValidator2018"
      }
    }
  }'
```

The `connectionId` identifies the established DIDComm connection to the holder. The `claims` object contains the credential's data as key-value pairs. The `issuingDID` is the issuer's published PRISM DID, and the `credentialSchema` references a schema previously created in the schema registry with the `id` field pointing to the schema's URL and `type` set to `JsonSchemaValidator2018`.

For SD-JWT credentials, the request uses `sdJwtVcPropertiesV1` instead:

```bash
curl -X POST 'http://localhost:8080/cloud-agent/issue-credentials/credential-offers' \
  -H 'Content-Type: application/json' \
  -H "apikey: $API_KEY" \
  -d '{
    "connectionId": "9d075518-f97e-4f11-9d10-d7348a7a0fda",
    "credentialFormat": "SDJWT",
    "sdJwtVcPropertiesV1": {
      "claims": {
        "emailAddress": "alice@wonderland.com",
        "givenName": "Alice",
        "exp": 1883000000
      },
      "issuingDID": "did:prism:9f847f8bbb66c112f71d08ab39930d468ccbfe1e0e1d002be53d46c431212c26",
      "credentialSchema": {
        "id": "http://localhost:8080/cloud-agent/schema-registry/schemas/...",
        "type": "JsonSchemaValidator2018"
      }
    }
  }'
```

SD-JWT credentials can include an `exp` claim (expiration as an epoch timestamp) that becomes a selectively disclosable field. The `issuingDID` must reference a DID using the Ed25519 curve.

For AnonCreds, the request uses `anoncredsVcPropertiesV1` and references a credential definition instead of a schema:

```bash
curl -X POST 'http://localhost:8080/cloud-agent/issue-credentials/credential-offers' \
  -H 'Content-Type: application/json' \
  -H "apikey: $API_KEY" \
  -d '{
    "connectionId": "9d075518-f97e-4f11-9d10-d7348a7a0fda",
    "credentialFormat": "AnonCreds",
    "anoncredsVcPropertiesV1": {
      "claims": {
        "emailAddress": "alice@wonderland.com",
        "givenName": "Alice",
        "drivingClass": "3"
      },
      "issuingDID": "did:prism:9f847f8bbb66c112f71d08ab39930d468ccbfe1e0e1d002be53d46c431212c26",
      "credentialDefinitionId": "5d737816-8fe8-3492-bfe3-1b3e2b67220b"
    }
  }'
```

AnonCreds claims must be flat string-to-string key-value pairs; nested objects are not supported. The `credentialDefinitionId` points to a previously created credential definition, which itself references a schema.

Once the offer is created, the Cloud Agent automatically delivers it to the holder via DIDComm messaging. The issuer can then wait for the holder to accept the offer, at which point the record transitions to the `RequestReceived` state.

If `automaticIssuance` is enabled (the default), the Cloud Agent will automatically issue the credential as soon as the holder's request is received. If set to `false`, the issuer must manually trigger the issuance with a separate API call:

```bash
curl -X POST "http://localhost:8080/cloud-agent/issue-credentials/records/$issuer_record_id/issue-credential" \
  -H "Content-Type: application/json" \
  -H "apikey: $API_KEY"
```

This endpoint can only be called when the record is in the `RequestReceived` state.

### Holder Flow

From the holder's perspective, the process begins when a credential offer arrives over DIDComm. The holder can list their credential records to see incoming offers:

```bash
curl "http://localhost:8090/cloud-agent/issue-credentials/records" \
  -H "Content-Type: application/json" \
  -H "apikey: $API_KEY"
```

To accept a JWT or SD-JWT offer, the holder provides their PRISM DID as the `subjectId`:

```bash
curl -X POST "http://localhost:8090/cloud-agent/issue-credentials/records/$holder_record_id/accept-offer" \
  -H 'Content-Type: application/json' \
  -H "apikey: $API_KEY" \
  -d '{
    "subjectId": "did:prism:subjectIdentifier"
  }'
```

For SD-JWT credentials with key binding, the holder can include a `keyId` parameter that binds the credential to a specific key:

```bash
curl -X POST "http://localhost:8090/cloud-agent/issue-credentials/records/$holder_record_id/accept-offer" \
  -H 'Content-Type: application/json' \
  -H "apikey: $API_KEY" \
  -d '{
    "subjectId": "did:prism:subjectIdentifier",
    "keyId": "key-1"
  }'
```

AnonCreds offers are accepted with an empty body, since the holder's identity is handled through the AnonCreds protocol rather than a subject DID:

```bash
curl -X POST "http://localhost:8090/cloud-agent/issue-credentials/records/$holder_record_id/accept-offer" \
  -H 'Content-Type: application/json' \
  -H "apikey: $API_KEY" \
  -d '{}'
```

After the holder accepts the offer, the Cloud Agent sends a credential request back to the issuer. Once the issuer processes the request and issues the credential, it is automatically delivered to the holder via DIDComm. The holder's record transitions to `CredentialReceived`, and the signed credential is stored in the agent's credential store, ready for use in future presentations.

## Revoking

Credential revocation allows an issuer to invalidate a previously issued credential. Once revoked, any presentation that includes the revoked credential will fail verification. This is a one-way operation; a revoked credential cannot be reinstated.

### Revocation Mechanism

Identus implements credential revocation according to the [W3C Verifiable Credentials Status List v2021](https://www.w3.org/TR/2023/WD-vc-status-list-20230427/) standard. Under this mechanism, every credential that supports revocation is issued with a `credentialStatus` property containing the information needed to check its revocation status:

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

The `statusListCredential` URL points to a Status List Credential, a special Verifiable Credential that contains a compressed bitstring where each bit position corresponds to a credential. The `statusListIndex` identifies this credential's position within that bitstring. When a credential is revoked, the issuer flips the corresponding bit from 0 to 1.

### Revoking a Credential

To revoke a credential, the issuer calls the revocation endpoint with the credential's record ID:

```
PATCH http://localhost:8080/cloud-agent/revoke-credential/<credential_id>
```

The issuer can find the `credential_id` by listing all issued credential records:

```bash
curl http://localhost:8080/cloud-agent/issue-credentials/records \
  -H "accept: application/json" \
  -H "apikey: $API_KEY"
```

And then revoking the target credential:

```bash
curl -X PATCH "http://localhost:8080/cloud-agent/revoke-credential/$credential_id" \
  -H "accept: */*" \
  -H "apikey: $API_KEY"
```

Only the issuer of a credential can revoke it.

### How Verification of Revocation Status Works

When a verifier receives a presentation containing a credential with a `credentialStatus` property, the verification process includes checking whether the credential has been revoked. This happens in four steps.

First, the verifier retrieves the Status List Credential from the URL specified in `credentialStatus.statusListCredential`. This credential is itself a Verifiable Credential and has the following structure:

```json
{
  "proof": { "..." },
  "@context": [
    "https://www.w3.org/2018/credentials/v1",
    "https://w3c.github.io/vc-bitstring-status-list/contexts/2021/v1.jsonld"
  ],
  "type": ["VerifiableCredential", "StatusList2021Credential"],
  "issuer": "did:prism:issuer_did",
  "issuanceDate": 1717793619,
  "credentialSubject": {
    "type": "StatusList2021",
    "statusPurpose": "Revocation",
    "encodedList": "BASE64_ENCODED_BITSTRING"
  }
}
```

Second, the verifier authenticates the Status List Credential by validating its embedded proof (digital signature). Identus supports two proof types for status list credentials: **DataIntegrityProof (eddsa-jcs-2022)** for Ed25519 keys, and **EcdsaSecp256k1Signature2019** for secp256k1 keys.

Third, the verifier decompresses the base64-encoded bitstring found in `credentialSubject.encodedList`.

Fourth, the verifier checks the bit at the position indicated by `statusListIndex`. If the bit is set to 1, the credential has been revoked. If it is 0, the credential remains valid.

::: {.callout-note}
The Status List approach has a notable privacy property: the verifier downloads the entire status list rather than querying the issuer about a specific credential. This means the issuer cannot learn which credential the verifier is checking, preserving the holder's privacy.
:::
