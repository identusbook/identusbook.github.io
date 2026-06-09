# DIDs and DID Documents {#sec-did-and-diddocuments}

## Overview

A **Decentralized Identifier (DID)** is a URI for a subject. The subject can be a person, organization, device, software agent, data model, or another entity chosen by the DID controller. The [W3C DID Core specification](https://www.w3.org/TR/did-1.0/) defines a DID as a URI with three parts: the `did:` scheme, a method name, and a method-specific identifier.

The DID itself is only the identifier. A resolver resolves the DID and returns a DID resolution result. A successful result includes a **DID Document**, resolution metadata, and DID Document metadata. The [W3C DID Resolution specification](https://www.w3.org/TR/did-resolution/) defines this resolver interface and leaves the method-specific retrieval steps to the DID method.

A **DID Document** publishes the public information software needs to interact with the DID subject: verification methods, verification relationships, and services. A DID Document should not contain private keys, secrets, or personal profile data. DID Core warns that public DID Documents can create privacy and correlation risk, so personal data belongs in credentials, peer-to-peer exchanges, or controlled service endpoints rather than the public document.

Identus developers use `did:prism` and `did:peer` for different jobs. `did:prism` fits public issuer and verifier identities that need ledger-backed resolution. `did:peer` fits relationship-specific DIDComm activity. A wallet, Cloud Agent, or mediator can use separate Peer DIDs to reduce correlation between relationships. The [Identus DID management documentation](https://hyperledger-identus.github.io/docs/home/identus/cloud-agent/did-management/) describes Cloud Agent support for both managed PRISM DIDs and managed Peer DIDs.

## DID Syntax and DID Methods

The generic syntax is:

```text
did:<method>:<method-specific-id>
```

For `did:prism`, the [PRISM DID method specification](https://github.com/input-output-hk/prism-did-method-spec/blob/main/w3c-spec/PRISM-method.md) defines this shape:

```text
did:prism:<initial-hash>[:<encoded-state>]
```

The short form contains the `did:prism:` prefix and a 64-character initial hash. The long form appends encoded initial state after another colon. PRISM uses the long form before the controller anchors the DID. After the controller publishes the DID operation and the PRISM VDR accepts it, software can use the short form as the published PRISM DID.

The encoded state in a long-form PRISM DID is not a JSON DID Document. It is method-specific state encoded by the PRISM method so a resolver can construct the initial DID Document before a public create operation appears in the VDR.

A **DID method** defines how software creates, resolves, updates, and deactivates DIDs and DID Documents for that method. DID Core defines the shared data model. The method specification defines the registry, protocol operations, encoding, validation rules, and lifecycle.

For `did:prism`, the PRISM method defines create, update, and deactivate operations. PRISM nodes read operations from the VDR and maintain the current state needed to construct DID Documents. The [PRISM VDR specification](https://github.com/hyperledger-identus/prism-vdr-driver/blob/main/prism-vdr-specification.md) describes SSI entries as event chains with create, update, and deactivate events.

## DID Subjects and Controllers

The DID identifies the DID subject. The DID method authorizes the DID controller to change the DID Document. These roles can be the same entity, but they do not have to be.

For example, a university can be the subject of `did:prism:...`. The university can run an Identus Cloud Agent that holds the controller keys and publishes DID updates through PRISM Node. The university is still the issuer in the credential exchange; the Cloud Agent is infrastructure acting for the university.

The top-level `controller` property in a DID Document names one or more controller DIDs. DID Core treats this authorization separately from `authentication`. A verifier that checks an authentication proof checks the `authentication` relationship. Software that processes DID update authority follows the DID method's controller rules.

The `controller` property inside a `verificationMethod` has a different scope. It identifies the controller of that verification method. It can match the DID Document `id`, or it can point to another DID when a system delegates key control.

## DID Document Structure

DID Documents share a common data model and can have different representations. DID Core and PRISM examples use JSON-LD with `@context`; JSON representations can omit JSON-LD context.

Common DID Document properties are:

| Property | What software uses it for |
| --- | --- |
| `id` | The DID of the subject described by the DID Document. |
| `controller` | One or more DIDs authorized to control the DID Document under the DID method's rules. |
| `verificationMethod` | Public verification material, such as JWKs, plus an `id`, `type`, and method controller. |
| `authentication` | Verification methods approved for challenge-response authentication as the DID subject. |
| `assertionMethod` | Verification methods approved for making claims, including issuing Verifiable Credentials. |
| `keyAgreement` | Verification methods approved for key agreement, including DIDComm encryption setup. |
| `capabilityInvocation` | Verification methods approved for invoking object capabilities. |
| `capabilityDelegation` | Verification methods approved for delegating object capabilities. |
| `service` | Endpoints or service descriptors for interaction, such as DIDComm messaging. |
| `@context` | JSON-LD context used by JSON-LD representations. |

Verification methods can be embedded directly in a verification relationship or referenced by DID URL. PRISM examples define methods once under `verificationMethod` and reference them from `authentication`, `assertionMethod`, or `keyAgreement`.

## Example PRISM DID Document

This example uses fake, shortened values so the structure stays readable. Do not treat these identifiers or keys as resolvable PRISM data.

```json
{
  "@context": [
    "https://www.w3.org/ns/did/v1",
    "https://w3id.org/security/suites/jws-2020/v1",
    "https://didcomm.org/messaging/contexts/v2"
  ],
  "id": "did:prism:example-university",
  "verificationMethod": [
    {
      "id": "did:prism:example-university#authentication-key-1",
      "type": "JsonWebKey2020",
      "controller": "did:prism:example-university",
      "publicKeyJwk": {
        "kty": "OKP",
        "crv": "Ed25519",
        "x": "fake-authentication-public-key"
      }
    },
    {
      "id": "did:prism:example-university#issuing-key-1",
      "type": "JsonWebKey2020",
      "controller": "did:prism:example-university",
      "publicKeyJwk": {
        "kty": "OKP",
        "crv": "Ed25519",
        "x": "fake-issuing-public-key"
      }
    },
    {
      "id": "did:prism:example-university#key-agreement-key-1",
      "type": "JsonWebKey2020",
      "controller": "did:prism:example-university",
      "publicKeyJwk": {
        "kty": "OKP",
        "crv": "X25519",
        "x": "fake-key-agreement-public-key"
      }
    }
  ],
  "authentication": [
    "did:prism:example-university#authentication-key-1"
  ],
  "assertionMethod": [
    "did:prism:example-university#issuing-key-1"
  ],
  "keyAgreement": [
    "did:prism:example-university#key-agreement-key-1"
  ],
  "service": [
    {
      "id": "did:prism:example-university#didcomm-1",
      "type": "DIDCommMessaging",
      "serviceEndpoint": [
        {
          "uri": "https://agent.example.edu/didcomm",
          "accept": ["didcomm/v2"],
          "routingKeys": ["did:peer:example-mediator#key-1"]
        }
      ]
    }
  ]
}
```

In the example, the issuer's software can use `#issuing-key-1` to sign a Verifiable Credential, and a verifier can check that the referenced key appears in `assertionMethod`. A DIDComm implementation can use `#key-agreement-key-1` to prepare encrypted messages and the `DIDCommMessaging` service to learn the accepted DIDComm profile, endpoint URI, and routing keys.

## DID Lifecycle

Every DID method defines its own lifecycle rules. The common operations are create, resolve, update, and deactivate. Identus developers see those operations through Cloud Agent APIs, resolver APIs, SDK resolver configuration, and PRISM Node behavior.

For a managed PRISM DID in Cloud Agent, the controller creates a DID from a document template. Cloud Agent derives PRISM key material from a wallet seed and derivation path, stores the derivation path, and can reconstruct key material at runtime from the seed. The Identus Quick Start creates a long-form PRISM DID, publishes it through Cloud Agent, and then uses the short form as `publishedPrismDID` for schema and credential flows.

PRISM updates publish method operations that change the DID state, such as adding or removing verification methods or services. The VDR state machine treats deactivation as a final state. After deactivation, resolvers should no longer treat the DID as active.

For a managed Peer DID, Cloud Agent generates random key material and stores it in secret storage. Peer DIDs are not used as global public issuer identifiers. They fit DIDComm relationships and mediation, where each relationship can use separate identifiers and keys.

## Resolvers

A resolver takes a DID and returns the DID Document plus metadata. The resolver must understand the DID method. A generic resolver can route different methods to method-specific drivers, but the method driver still performs the method-specific retrieval and validation.

For `did:prism`, a resolver needs the current PRISM state for the DID. Identus documents these resolution paths: Cloud Agent with PRISM Node, Universal Resolver support for PRISM DIDs, SDK resolver configuration, and community indexers. The [Identus DID PRISM Resolver documentation](https://hyperledger-identus.github.io/docs/home/identus/did-prism-resolver/) describes Cloud Agent and PRISM Node as a solution for creating, updating, deactivating, and resolving PRISM DIDs.

Resolver choice is a trust decision. A verifier can use a hosted resolver, run its own resolver, or resolve against local indexed state. The verifier software should treat resolver output as input to verification, not as the whole trust decision. Credential verification still checks the proof, the verification relationship, credential status, schema, and verifier policy.

## Controllers

Controllers are entities authorized to change the DID Document under the DID method's rules. A controller can be a person using a wallet, an organization running Cloud Agent, or a service acting under an organization's operational control. The DID Document records controllers as DIDs; the legal or governance authority behind those controllers lives outside the DID Document.

In `did:prism`, the controller proves authority through the method's signed operations. PRISM VDR validation rules require create operations to be signed with the DID's master key and update or deactivate events to be signed by a current master key. Cloud Agent-managed PRISM DID APIs perform this protocol work for the application, but the controller model still determines who can rotate keys, add services, or deactivate the DID.

## Verification Flow

A verifier that checks a credential issued from a PRISM DID uses this flow.

1. The verifier receives a presentation from the holder's wallet or Edge Agent.
2. The verifier extracts the issuer DID and the verification method referenced by the credential proof.
3. The resolver resolves the issuer DID to a DID Document.
4. The verifier dereferences the verification method and checks that the method appears under `assertionMethod`.
5. The verifier checks the cryptographic proof with the public key in the verification method.
6. The verifier checks credential status, schema, and policy, including whether the issuer is trusted for the requested transaction.

DID verification relationships prevent key reuse across proof purposes. A key listed only under `authentication` cannot be used to issue a credential. A key listed only under `assertionMethod` cannot be used for DIDComm key agreement. The verifier software has to check the relationship that matches the operation it is validating.

## Privacy Checks

Public DIDs are useful for issuers, verifiers, schemas, and trust registries, but they create correlation. A public issuer DID should remain stable so verifiers can resolve issuer keys and policy systems can refer to the issuer. A holder's wallet should use pairwise or relationship-specific DIDs for DIDComm connections unless the holder intends public correlation.

Before publishing a DID Document, the controller should check that it contains only the public material needed for the DID method and protocol flows:

1. No private keys, seeds, bearer tokens, credentials, or personal profile attributes.
2. No service endpoint URL that embeds a username, customer identifier, account number, or other correlating value.
3. No reused verification method across relationships intended to stay unlinkable.
4. No descriptive `type` or custom property that reveals the subject's category without a protocol need.

DIDs stay focused on public verification and routing. Credentials and presentations carry claims. Agents and wallets handle disclosure to specific verifiers.
