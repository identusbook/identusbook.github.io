# SSI Basics {#sec-ssi-basics}

## Introduction to Self-Sovereign Identity

Self-Sovereign Identity (SSI) represents a paradigm shift in how we manage digital identity. Unlike traditional centralized identity systems where third parties control and store our personal data, SSI puts individuals in control of their own identity information.

Imagine your digital life today: you have dozens of accounts across various websites and services, each with different login credentials and each storing pieces of your personal information. When you apply for a loan, you must provide the same information repeatedly. If a service is breached, your data is exposed. And for over a billion people worldwide without formal identification, accessing basic services remains impossible.

SSI addresses these challenges by creating a decentralized identity layer for the internet where individuals own and control their identity data, sharing only what's necessary with whom they choose.

## Core Concepts of SSI

To begin understanding SSI and how it works, these are the fundamental building blocks that you must get familiar with, we will explore in depth each one of these concepts and how to use them in Identus as we progress trough the book.

### Decentralized Identifiers (DIDs)

DIDs are globally unique identifiers that enable verifiable, decentralized digital identity. Think of a DID as a digital address that belongs exclusively to you—not assigned by Facebook or Google, but created and controlled by you.

For example, when Alice creates her digital identity in an SSI ecosystem, she generates a DID like `did:prism:4a9bce8d72e4c30017c42f2b` that she controls through cryptographic keys on her smartphone.

### DID Documents

A DID Document contains the information payload associated with your DID. It contains public keys for verification and might include service endpoints, for example, with descriptions of ways to communicate with the DID owner, a URL to an asset, etc.

When a bank wants to verify Alice's identity, it can use her DID Document to establish secure communication and verify her credentials without relying on a central authority.

### Verifiable Credentials (VCs)

Verifiable Credentials are digital equivalents of physical credentials. Just as your physical wallet might contain a driver's license, university diploma, and health insurance card, your digital wallet contains VCs that prove various aspects of your identity.

For instance, Alice's digital wallet might contain a VC issued by her university confirming her degree, another from her employer verifying her employment, and one from her government confirming her citizenship, all cryptographically secured and under her control.

### Verifiable Presentations (VPs)

A Verifiable Presentation allow a holder to interchange cryptographic proof with a verifier about a particular VC. Depending on the format of the Verifiable Credencial, the proof could be about the whole credential or a subset of particular claims. For example, during a regular Verifiable Presentation of a standard W3C JWT VC, the whole credential is presented plus a signed challenge as proof as opposed with a SD-JWT VC, where only a selective disclosure of information from the credential is presented. So when Alice applies for a job, she doesn't need to share her entire identity, just relevant qualifications. Using a VP, she can prove she has a computer science degree without revealing her birth date or address that might also be in that credential.

## The Triangle of Trust

The SSI ecosystem operates through what's commonly called the "Triangle of Trust", a relationship between three primary roles in any given interaction. Keep in mind that any person or entity can fulfill each role depending on the context of the interaction, for example, at some point you may receive a credential, this makes you the holder in that interaction, and later that day you may want to verify someone's credential, that would make you a verifier in that interaction. This is one of the first insights that you need to internalize.

### Issuer

The entity that issues a Verifiable Credential to a Holder. This could be a bank, government agency or anyone that accepts responsibility for making credentials. For example a governmental agency can issue a passport to a citizen, or a gym can issue a membership to a member. The type of agency is not important, only that they issue a credential. This role accepts responsibility for having issued that credential and should look to establish trust and reputation with Holders and Verifiers. 

For example, when the Department of Motor Vehicles issues Alice a digital driver's license, it acts as an Issuer, cryptographically signing a credential that contains her driving privileges and personal information.

The credential includes the DMV's DID in the Issuer field, allowing anyone to verify its origin. The DMV maintains its reputation by issuing accurate credentials and properly verifying identities before issuance.

### Holder

The Holder receives and stores credentials. Alice becomes a Holder when she receives her digital driver's license in her wallet app. She controls this credential completely—deciding when, how, and with whom to share it.

When a police officer asks for identification during a traffic stop, Alice can present her digital license through her wallet. The wallet application handles the cryptographic proof that she is indeed the rightful owner of this credential.

### Verifier

A Verifier is any person or entity that performs a verification on a Verifiable Credential or any of its referenced entities. A Verifier might perform a check on cryptographic elements of a VC, or make sure that the Issuer DID belongs to the expected entity. Verifiers usually only care to double check that the Verifiable Credential is legitimate and that the included claims meet their expectations.

Following our previous examples, when Alice uses her digital driver's license at a bar, the bartender becomes a Verifier. The bartender's verification app confirms that:

- The credential was issued by a legitimate DMV (by checking the Issuer's DID)
- The credential hasn't been tampered with (through cryptographic verification)
- The credential hasn't been revoked (by checking a revocation registry)
- Alice is the rightful owner (through cryptographic proof)
- Alice is over 21 (by reading the birthdate claim in the credential)

All this happens digitally in seconds, with greater security than visual inspection of a physical ID.

### Trust Registry

When Verifiers need to know who a DID belongs to, there needs to be a way to look up that information. A Trust Registry is a mapping between DIDs and the entities they represent. You can think of this like a phone book for DIDs. Trust Registries need to be trusted themselves, and can be as broad or as specific as they need to be. For example, a Real Estate specific Trust Registry that lists real estate agents can be a way to validate that a particular agent is an accepted member of that industry.

In essence, a Trust Registry provides the foundation for knowing which issuers to trust. When the bartender's app verifies Alice's driver's license, it consults a Trust Registry to confirm that the DID belongs to the actual DMV and not an impostor.

For example, a national Trust Registry might list all authorized government agencies and their DIDs, while an industry-specific registry might list accredited universities or professional certification bodies.