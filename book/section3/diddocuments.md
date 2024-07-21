---
title: "DID Documents"
---
## Overview

A DID (Decentralized Identifier) Document is a JSON-LD (JavaScript Object Notation for Linked Data) structure which describes a DID Subject. This can represent the identity of a person or a thing.  Contained in the document is information which can verify that identity without relying on a centralized authority.

The spec for a `did:prism` DIDDocument can be found [here](https://github.com/input-output-hk/prism-did-method-spec/blob/main/w3c-spec/PRISM-method.md#did-documents).

Components of a DID Document:

- **DID**: The DID of the Subject described by the DIDDocument
- **Context**: Labled as `@context`, this is an array of context specs for the DIDDocument.  The first element is usually [https://www.w3.org/ns/did/v1](https://www.w3.org/ns/did/v1) but any other standards used in the DIDDocument should be listed here, for example URIs to definitions for [JSONWebSignature](https://w3id.org/security/suites/jws-2020/v1) or [DIDComm2 Messaging](https://didcomm.org/messaging/contexts/v2) protocol.
- **Verification Method**: An array of information which can be used to verify the identity of the Subject.
    - `id`: The DID of the Subject
    - `controller`: The DID of the Subject (author's note: When could this be different than id?)
    - [publicKeyJwk](https://www.w3.org/TR/did-core/#dfn-publickeyjwk) or [publicKeyMultibase](https://www.w3.org/TR/did-core/#dfn-publickeymultibase): 
        - `publicKeyJwk`: A JSON Web Key (JWK) representation of the Subject's Public Key
        - `publicKeyMultibase`: An encoded public key using [Multibase](https://www.ietf.org/archive/id/draft-multiformats-multibase-08.html) encoding
    - `type`: The type of [Verification Method](https://www.w3.org/TR/did-core/#dfn-verification-method), ie [`Ed25519VerificationKey2020`](https://www.w3.org/community/reports/credentials/CG-FINAL-di-eddsa-2020-20220724/#ed25519verificationkey2020) or [JsonWebKey2020](https://www.w3.org/community/reports/credentials/CG-FINAL-lds-jws2020-20220721/#json-web-key-2020)
- **Authentication Methods**:
    -   `authentication`, `assertionMethod`, `keyAgreement`: Arrays of locations in the Subject DID, referenced in a DID + anchor format (`did:prism:1234#authentication0)
    - *Author's note - Specify these in a more concrete way
- **Services**: An array of `service` descriptions, including URLs that can be used to further interact with the DID Subject. These could be API endpoints, for messaging or file storage systems, but any remote service can be added to add value to the DID
- **Controller**: An array of DIDs that are allowed to mutate the DIDDocument
- **Proofs**: (*author's note: TBD) Contains cryptographic proofs that assert the validity and integrity of the DID Document, ensuring it has not been tampered with.

An non-exhaustive example of a `did:prism` DIDDocument can be found [here](https://github.com/input-output-hk/prism-did-method-spec/blob/main/w3c-spec/PRISM-method.md#example-did-document-json-ld).

## Resolvers

## Controllers