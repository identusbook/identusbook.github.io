# DIDs and DID Documents {#sec-did-and-diddocuments}

## Overview

A **DID** (*Decentralized Identifier*) is the canonical representation of a DIDDocument; a portable, compact hash, which can be passed around easily or stored to a database or blockchain. A DID can be *resolved*, revealing the full, parsable JSON encoded DIDDocument.

A **DID Document** (*Decentralized Identifier Document*) is a JSON-LD (*JavaScript Object Notation for Linked Data*) structure which describes a Subject. This can represent the identity of a person, a thing, or a relationship between one or many entities. Contained in the document is information which can verify that identity without relying on a centralized authority.

TODO: Add DID Method overview
A **DID Method** ..


The spec for a `did:prism` DIDDocument can be found [here](https://github.com/input-output-hk/prism-did-method-spec/blob/main/w3c-spec/PRISM-method.md#did-documents).

An Example DID:
`
did:prism:4a5b5cf0a513e83b598bbea25cd6196746747f065246f1d3743344b4b81b5a74:Cr4BCrsBElsKBmF1dGgwMRJRCglzZWNwMjU2azESBHRlc3QaOmtleTE6Ly8wMjM5MmYxNjc4NmNlNmQ0NzJlOGViNzA4ZWRjMmE3OTFmZGMxNzNkNjVkNTBhODNhMTk3N2I5ZmIwMmU0MjQSWwoGYXV0aDAyElEKCXNlY3AyNTZrMRIEdGVzdBo6a2V5MjovLzAyMzkyZjE2Nzg2Y2U2ZDQ3MmU4ZWI3MDhlZGMyYTc5MWZkYzE3M2Q2NWQ1MGE4M2ExOTc3YjlmYjAyZTQyNA
`

Let's break down the format of this example DID:

- `did:prism:` The prefix of the DID
- `4a5b5cf0a513e83b598bbea25cd6196746747f065246f1d3743344b4b81b5a74`: The DID identifier.  This can be anything, as long as it is unique to the DID Document it is describing, and means something to your application.
- `Cr4BCrsBElsKBmF1dGgwMRJRCglzZWNwMjU...`: The DID Document itself, encoded in base58

An Example DIDDocument:

```json
{
  "@context": [
      "https://www.w3.org/ns/did/v1",
      "https://w3id.org/security/suites/jws-2020/v1",
      "https://didcomm.org/messaging/contexts/v2",
      "https://identity.foundation/.well-known/did-configuration/v1"
    ],
  "id": "did:prism:123456789abcdefghi",
  "controller": "did:example:bcehfew7h32f32h7af3",
  "verificationMethod": [{
    "id": "did:prism:123456789abcdefghi#key-1",
    "type": "JsonWebKey2020",
    "controller": ["did:prism:123456789abcdefghi"],
    "publicKeyJwk": {
      "kty": "OKP",
      "crv": "Ed25519",
      "x": "VCpo2LMLhn6iWku8MKvSLg2ZAoC-nlOyPVQaO3FxVeQ"
    }
  }],
  "authentication": ["did:prism:123456789abcdefghi#key-1"],
  "assertionMethod": ["did:prism:123456789abcdefghi#key-1"],
  "keyAgreement": [ "did:prism:123456789abcdefghi#key-1"],
  "service": [{
    "id": "did:prism:123456789abcdefghi#messaging",
    "type": "DIDCommMessaging",
    "serviceEndpoint": "https://example.com/endpoint"
  }]
}
```

Let's look at the components of a DID Document:

- **`id`**: The DID of the Subject described by the DIDDocument
- **`@context`**: This is an array of specifications used in this DIDDocument.  The first element is usually [https://www.w3.org/ns/did/v1](https://www.w3.org/ns/did/v1) but any other common definitions are [JSONWebSignature](https://w3id.org/security/suites/jws-2020/v1) or [DIDComm2 Messaging](https://didcomm.org/messaging/contexts/v2) protocols.
- **`controller`**: An array of DIDs that are allowed to mutate the DIDDocument
- **`verificationMethod`**: An array of information which can be used to verify the identity of the Subject.
    - `id`: The DID of the Subject
    - `controller`: The DID of the Subject (author's note: When could this be different than id?)
    - [publicKeyJwk](https://www.w3.org/TR/did-core/#dfn-publickeyjwk) or [publicKeyMultibase](https://www.w3.org/TR/did-core/#dfn-publickeymultibase): 
        - `publicKeyJwk`: A JSON Web Key (JWK) representation of the Subject's Public Key
        - `publicKeyMultibase`: An encoded public key using [Multibase](https://www.ietf.org/archive/id/draft-multiformats-multibase-08.html) encoding
    - `type`: The type of [Verification Method](https://www.w3.org/TR/did-core/#dfn-verification-method), ie [`Ed25519VerificationKey2020`](https://www.w3.org/community/reports/credentials/CG-FINAL-di-eddsa-2020-20220724/#ed25519verificationkey2020) or [JsonWebKey2020](https://www.w3.org/community/reports/credentials/CG-FINAL-lds-jws2020-20220721/#json-web-key-2020)
- **Authentication Methods**:
    -   `authentication`, `assertionMethod`, `keyAgreement`: Arrays of locations in the Subject DID, referenced in a DID + anchor format (`did:prism:1234#authentication0`)
    - *Author's note - Specify these in a more concrete way
- **`service`**: An array of advertised methods of interacting with the Subject. These could be API endpoints for messaging or file storage systems, but any remote service can be added to add value to the DID.

An non-exhaustive example of a `did:prism` DIDDocument can be found [here](https://github.com/input-output-hk/prism-did-method-spec/blob/main/w3c-spec/PRISM-method.md#example-did-document-json-ld).

## DID Lifecycle

TODO: Describe the lifecycle of DIDs (creation, update, deactivation)

## Resolvers

A resolver is a service that can resolve a DID to a DIDDocument.  There are PRISM specific resolvers built into Identus SDKs, or you can also run your own resolver service.

Some third-party PRISM resolvers:

- [Blocktrust Resolver](https://analytics.blocktrust.dev/resolve)
- [NeoPrism Resolver](https://neoprism.patlo.dev/resolver)

## Controllers

**Controllers** are entities that can mutate the DIDDocument. Controllers are specified in the DIDDocument as an array of DIDs so they can be a person, thing, or organization.

Remember that DIDs can all be resolved to DIDDocuments, and each DIDDocument can point to people, things, machines, or services. Every mention of a DID can potentially be a chain of references to other services, or endpoints.  There is plenty of room to be creative with this relationship graph.