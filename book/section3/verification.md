# Verification {#sec-verification}

## Presenting proof

Verification in Identus starts when a verifier asks a holder to present evidence from one or more credentials. For DIDComm flows, Identus Cloud Agent uses the Present Proof protocol. The verifier Cloud Agent sends a presentation request over an existing DIDComm connection, the holder Cloud Agent returns a presentation, and the verifier Cloud Agent checks the received proof material.

The [Identus Present Proof guide](https://github.com/hyperledger-identus/cloud-agent/blob/main/docs/docusaurus/credentials/didcomm/present-proof.md) describes the DIDComm API flow:

1. The verifier controller calls `POST /present-proof/presentations` on its Cloud Agent. The request identifies the DIDComm connection and describes the proof the verifier wants.
2. The holder controller reads pending presentation records from `GET /present-proof/presentations`.
3. The holder controller accepts a request with `PATCH /present-proof/presentations/{id}` using `action: "request-accept"`. For JWT and SD-JWT credentials it supplies credential record IDs through `proofId`. For AnonCreds it supplies the credential proof mapping under `anoncredPresentationRequest`.
4. The holder Cloud Agent generates the presentation and sends it to the verifier Cloud Agent.
5. The verifier Cloud Agent verifies the presentation. A successful verifier-side record moves through `PresentationReceived` and `PresentationVerified`.
6. The verifier controller accepts the verified presentation with `PATCH /present-proof/presentations/{id}` using `action: "presentation-accept"`, or rejects it in application logic.

The exact request shape depends on the credential format. JWT and SD-JWT presentation requests can include `options.challenge` and `options.domain` so the holder signs material bound to the verifier's request. AnonCreds presentation requests use an `anoncredPresentationRequest` object with a nonce, requested attributes, requested predicates, restrictions, and optional non-revocation intervals.

The [DIDComm Present Proof 3.0 protocol](https://didcomm.org/present-proof/3.0/) is format-neutral. It carries a presentation request and a presentation, but the credential format decides which cryptographic checks run.

## Verification and acceptance

Cryptographic verification and verifier acceptance are separate decisions.

The verifier Cloud Agent can check whether the presentation is well formed for its format, bound to the verifier request, and backed by the expected cryptographic material. For JWT-VC, the verifier checks JWT signatures and DID-resolved keys. For SD-JWT-VC, the verifier checks the issuer signature, disclosed claim digests, and holder key binding when the credential carries a `cnf` key. For AnonCreds, the verifier checks equality proofs, predicate proofs, aggregate proofs, and any non-revocation proofs requested by the verifier.

The verifier controller still decides whether the proof should be accepted for the business action. The [W3C Verifiable Credentials Data Model 2.0](https://www.w3.org/TR/vc-data-model-2.0/) separates verification from validation: verification checks the securing mechanism, and validation applies business rules to a specific use of the credential.

After Cloud Agent reports a verified presentation, the verifier controller should check:

1. The verifier trusts the issuer DID for the credential type and schema.
2. The credential uses the schema or credential definition named in the verifier's request.
3. The disclosed claims satisfy the verifier's policy.
4. The credential validity period and any status or revocation data fit the verifier's time of interest.
5. The verifier accepts the holder or presenter as authorized for this use case.

A valid proof can still fail acceptance. A venue verifier can receive a cryptographically valid age credential, reject it if the issuer is outside its trusted issuer list, and reject it if the revealed claim does not meet the venue's age rule.

## Direct credential verification

Cloud Agent exposes a direct verification endpoint at `POST /verification/credential`. Use this endpoint when verifier software has an encoded JWT credential and needs explicit check results outside a DIDComm present-proof exchange.

The endpoint accepts a list of encoded credentials and verification checks. The current service implementation includes checks such as signature verification, algorithm verification, issuer identification, expiration, not-before, audience, schema validation, subject schema validation, and semantic parsing of claims.

The direct verification endpoint does not replace a presentation request. It verifies credential data supplied to the endpoint. It does not prove that the current holder controls the credential subject's key for a verifier challenge, and it does not decide whether an issuer or claim should be accepted by the application.

## Verification policies

Identus uses the term `verification policy` for stored verifier rules. The Cloud Agent source exposes `/verification/policies` endpoints to create, update, fetch, list, and delete those policies. The current policy model contains a `CredentialSchemaAndTrustedIssuersConstraint` with:

1. `schemaId`, the schema the verifier expects.
2. `trustedIssuers`, the issuer DIDs the verifier accepts for that schema.

The endpoint description in the Cloud Agent source says these policies apply to W3C Verifiable Credentials in JWT format and currently use `schemaId` and `trustedIssuers` constraints.

A verifier controller can use a verification policy before sending a presentation request, after receiving a verified presentation, or both. Before the request, the policy can help the verifier ask for the schema it accepts. After verification, the policy can help the verifier reject a credential from an untrusted issuer. The Cloud Agent's cryptographic verification result should feed into this policy decision; it should not be treated as the whole decision.

If a deployment uses an external trust registry, keep the registry lookup separate from proof verification. The verifier or its controller queries the registry, resolves the trusted issuer set for the requested schema, and passes that result into its policy decision.

## Selective disclosure

Selective disclosure lets a holder reveal only the claims needed for a verifier request. Identus supports selective-disclosure presentations through SD-JWT and AnonCreds, but the two formats use different mechanisms.

Plain JWT-VC presentation exposes the credential payload needed for verifier checks. The format works for use cases where the verifier needs the full credential, such as a boarding gate checking a complete boarding pass. It fits poorly when the verifier needs one attribute or one predicate.

SD-JWT uses salted hashes. The issuer signs a JWT payload that contains digests for selectively disclosable claims. During presentation, the holder sends the issuer-signed SD-JWT plus the disclosures selected for that verifier. The verifier hashes each disclosed value with its salt and checks that the digest appears in the signed payload. SD-JWT gives selective disclosure for JSON claims; it is not a zero-knowledge predicate proof system.

Identus adds one key-binding detail for SD-JWT presentations. If the holder accepts an SD-JWT credential offer with a `keyId`, the issued credential can include a `cnf` key binding. A later holder presentation can then sign the verifier's `challenge` and `domain`. If the SD-JWT credential has no `cnf` key, the Identus Present Proof guide says the holder cannot create a presentation that signs the verifier's `challenge` and `domain`.

AnonCreds uses zero-knowledge proofs. The verifier's presentation request can ask for revealed attributes, predicates such as `age >= 18`, restrictions such as `schema_id` or `cred_def_id`, and non-revocation intervals. The holder selects credentials that satisfy the request and creates a proof. The verifier resolves the required schemas, credential definitions, and revocation data, then verifies the presentation.

AnonCreds lets the holder prove a predicate without revealing the raw attribute. For age verification, a government issuer can issue a credential containing a date-derived age attribute. An age-restricted venue can request an AnonCreds predicate for `age >= 21`. The holder presents a proof that satisfies the predicate, and the verifier checks the proof, issuer restriction, credential definition, and revocation interval without receiving the holder's full date of birth.

Selective disclosure does not remove verifier policy. The verifier still checks issuer trust, schema fit, credential status, request binding, and local acceptance rules for the disclosed claims or predicates.
