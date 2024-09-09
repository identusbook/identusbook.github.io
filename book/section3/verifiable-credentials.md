---
title: "Verifiable Credentials"
---

## Overview

Verifiable Credentials are an integral part of Self-Sovereign Identity, allowing individuals to keep and control how their personal information is shared.

A Verifiable Credential (VC) is a digital statement made by an **Issuer** about a **Subject**. This statement is cryptographically secured and can be verified by a third party without the need for the **Verifier** to directly contact the **Issuer**. Verifiable Credentials are used to represent information such as identity documents, academic records, professional certifications, and other forms of credentials that traditionally exist in paper form.  Coupled with other technology such as Self-Sovereign Identity, Verifiable Credentials can unlock novel and exciting use cases.

Components of a Verifiable Credential:

- **Issuer**: The entity that creates and signs the credential. This could be an organization, institution, government entity or individual.
- **Holder**: The entity or individual to whom the credential is issued to and who can present proof to a **Verifier**.
- **Verifier**: The entity or individual that checks the authenticity and validity of the credential trough requesting proof from a **Holder**.
- **Subject**: The entity or individual about which the claims are made. In many cases, the **Holder** and the **Subject** are the same entity.
- **Claims**: Statements about the **Subject**, such as "Alice has an Educational Credential from Vienna University."
- **Proof**: Cryptographic evidence, using Digital Signatures, that the credential is authentic and has not been tampered with.
- **Metadata**: Additional contextual information which may have content or application specific meaning, like expiration date or credential description.

How Verifiable Credentials Work:

- **Issuance**: The issuer creates a credential containing claims about the subject, the subject DID (public key) and signs it with their private key.
- **Storage**: The holder receives the credential and stores it in a digital wallet.
- **Presentation**: When required, the holder presents the credential to a verifier. Selective Disclosure can be used to reveal only relevant context about a claim, and not all user data.
- **Verification**: The verifier checks the credential’s authenticity by validating the issuer’s digital signature and ensuring the credential has not been tampered with.

Benefits of Verifiable Credentials:

- **Interoperability**: VCs follow standard formats, making them compatible across different systems and platforms.
- **Privacy**: Holders can share only the necessary information, protecting their privacy.
- **Security**: Cryptographic techniques ensure the integrity and authenticity of credentials.
- **Decentralization**: VCs do not rely on a central authority for verification, reducing single points of failure.

Use Cases:

- **Digital Identity**: Proof of identity for accessing services.
- **Education**: Digital diplomas and certificates.
- **Healthcare**: Vaccination records and medical certificates.
- **Employment**: Professional qualifications and work experience.

## Formats

There are several formats for Verifiable Credentials, including:

- [W3C - v1.1](https://www.w3.org/TR/vc-data-model/)
- [W3C - v2.0](https://www.w3.org/TR/vc-data-model-2.0/) (*Atala Roadmap*)
- [SD-JWT VC - Active Internet-Draft](https://datatracker.ietf.org/doc/draft-ietf-oauth-sd-jwt-vc/)
- [OID4VCI](https://openid.net/2023/02/22/oid4vci-1-0-release/) (*Atala Roadmap?*)
- [AnonCreds](https://github.com/hyperledger/anoncreds-spec)

## Schemas

Issuing a Verifiable Credential (VC) requires a credential schema, which serves as a general template defining the valid claims (attributes) the VC can contain. This schema acts as a reference point to ensure that the VC is correctly formatted and valid by checking its claims against the predefined structure.

Schemas can optionally be published on a Verifiable Data Registry (VDR), which is particularly beneficial for widely applicable schema types. Publishing schemas on a VDR facilitates their adoption by other parties, enabling third parties to issue VCs that conform to the same standardized credential format.

For example, relevant entities or industry consortiums can collaboratively develop, agree upon, and publish schemas that they will adopt and recognize. This approach encourages other players in the ecosystem to adopt these schemas as well, fostering interoperability and growth within the ecosystem.

By standardizing schemas, the VC ecosystem becomes more cohesive and efficient, allowing for easier verification and broader acceptance of credentials across different platforms and organizations.

Example Credential Schema
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

- Publishing your Schema (Milestone 3)

## Issuing

Issuing a Verifiable Credential (VC) is a multi-step process that occurs between an issuer agent and a holder. Currently, this process is only supported through the cloud agent's API endpoints, as there is no functionality to issue VCs from edge client SDKs.

The issuing process shares a common prerequisite across all three supported VC formats (JWT, SD-JWT, and AnonCreds): an established connection between the issuing cloud agent and a holder. The holder can be either another cloud agent or an edge client device.

Additional requirements vary depending on the VC format:

1. For JWT and SD-JWT:
   - The issuing agent must have a published DID Prism.

2. For SD-JWT only:
   - The holder must also have a DID Prism, but it doesn't need to be published on-chain.

3. For AnonCreds:
   - No additional requirements beyond the established connection.

### Issuer flow

From the issuer perspective this is the regular flow to issue a VC:

1. Create a credential offer over API endpoint
2. Send the credential offer to holder over DIDCOMM
3. Receive credential request from holder over DIDCOMM
4. Issue and process credential
5. Send credential to holder over DIDCOMM
   
Depending on the value of `automaticIssuance` the credential will be automatically issued on step 4 as soon as the credential request is received from the holder, if `automaticIssuance` is set to `false`, the issuer must manually trigger issuance and process trough an API call. 

### Holder flow

From the holder perspective this is regular flow to receive a VC:

1. Received offer over DIDCOMM
2. Accept offer, in this step the SDK calls cloud agent API endpoint to trigger the issuance of the VC
3. Receive credential over DIDCOMM

TODO: Add code / API calls on Milestone 3.

## Revoking

Revoking a VC is done through a simple API call to the cloud agent to revoke a specific credential by it's ID. The end result is that any presentation proof request will fail if the VC has been revoked.

TODO: Add code / API calls on Milestone 3.