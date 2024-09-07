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
    - [SD-JWT - O2](https://www.ietf.org/archive/id/draft-fett-oauth-selective-disclosure-jwt-02.html)
    - [SD-JWT - 04](https://datatracker.ietf.org/doc/draft-ietf-oauth-sd-jwt-vc/)
    - [OID4VCI](https://openid.net/2023/02/22/oid4vci-1-0-release/) (*Atala Roadmap?*)
    - [AnonCreds](https://github.com/hyperledger/anoncreds-spec)

## Schemas

- Schemas
- Publishing your Schema

## Issuing

- JWT
- SD-JWT (Atala Roadmap Q2)
- AnonCreds

## Updating

- Updating (re-binding)

## Revoking

- Revoking

