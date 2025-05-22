# Verifiable Credentials {#sec-verifiable-credentials}

## Overview

Verifiable Credentials (VCs) are a cornerstone of Self-Sovereign Identity (SSI), empowering individuals to manage and control the sharing of their personal information. Throughout this book, we will explore VCs by building an example airline ticket wallet, where Verifiable Credential serves a digital flight ticket, illustrating the concepts in a practical manner.

At its core, a Verifiable Credential is a digital assertion made by an **Issuer** concerning a **Subject**. This assertion is cryptographically secured, enabling a **Verifier** to confirm its authenticity and integrity without needing to directly contact the Issuer. VCs can represent a wide array of information, from identity documents and academic degrees to professional certifications and, as in our project, airline tickets. They offer a digital, secure, and verifiable alternative to traditional paper-based credentials, unlocking innovative applications when combined with SSI principles.

Understanding VCs involves recognizing their key components. The **Issuer** is the entity that creates and cryptographically signs the credential; in our airline example, this is the Airline company responsible for issuing flight tickets. The **Holder** is the individual or entity to whom the credential is issued and who controls its presentation; this role is filled by the passenger who has purchased the ticket. When the passenger needs to prove their right to board a flight, they present their ticket to a **Verifier**, in this case, the Airport Security Officer, who then checks the credential's authenticity and validity. The **Subject** is the entity the credential's claims are about. Keep in mind that very often, as with our example, the Holder and the Subject are one and the same, in our case is the passenger. The credential contains **Claims**, which are specific statements about the Subject, such as the passenger's name, flight number, and seat assignment for an airline ticket. The security and trustworthiness of a VC are guaranteed by its **Proof**, typically a digital signature, which provides cryptographic evidence of authenticity and integrity. Finally, **Metadata** offers additional context, such as the credential's expiration date or a description of its purpose, like "Airline Boarding Pass."

The lifecycle of a Verifiable Credential involves several key stages, which we will implement in our example project:

The **Issuance** process is how a VC is created and delivered. As per the Identus Cloud Agent's implementation of the Issue Credential Protocol 3.0, this flow is initiated by the Issuer. For instance, the Airline (Issuer) would first create a credential offer—a proposal to issue a specific ticket—and send it to the passenger (Holder) via DIDComm. If the passenger accepts this offer, they respond with a credential request. The Airline then issues the Verifiable Credential containing claims like flight details and the passenger's DID. In the last step of issuance, the Issuer signs with their private key and sends it to the Holder.

Once received, the **Storage** phase begins. The passenger (Holder) stores this digital ticket securely in their digital wallet, such as the airline ticket wallet mobile application we will develop.

When the passenger passes trough security at the airport, a **Presentation** occurs. The passenger (Holder) presents their digital airline ticket to the Airport Security Officer (Verifier). This can involve presenting the entire credential or, through mechanisms like Selective Disclosure, only specific relevant claims (e.g., just the name and flight number, without revealing the ticket price or other details), thereby enhancing privacy.

Finally, **Verification** is performed by the Airport Security Officer. They use their systems to check the digital signature on the ticket to confirm its authenticity (i.e., it was indeed issued by the legitimate Airline) and integrity (i.e., it has not been tampered with since issuance). This step ensures that the ticket is valid and trustworthy.

A note on **Trust**. Just because a Verifiable Credential is legitimate and passes the cryptographic verification, it doesn't mean the presentation will be accepted. There might be other business rules and checks needed to establish a final decision. For example, the passenger might be part of a "no fly" list or lack a required visa for international travel, and even though it has a legitimate ticket, the Airport Security Officer might reject the presentation. The important insight is that after the presentation reaches the verified status, there needs to a **manual** final decision to accept or reject a presentation.

In general, Verifiable Credentials offer significant advantages. Their adherence to standard formats promotes **Interoperability**, ensuring they can be used across diverse ecosystems and platforms without vendor lock-in. **Privacy** is also a key benefit, as Holders can selectively disclose only the necessary information, rather than revealing all data contained within a credential, giving them granular control over their personal data. The use of cryptographic techniques provides robust **Security**, guaranteeing the integrity and authenticity of credentials against tampering and forgery. Furthermore, VCs used within Self-Soverign Identity interactions support **Decentralization**, as verification often doesn't require real-time communication with the original Issuer or reliance on a central authority, thereby reducing single points of failure and control.

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
