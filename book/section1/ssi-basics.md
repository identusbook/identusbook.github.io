---
title: "SSI Basics"
---

[SSI Roles/Conceptual Diagram (Triangle of Trust)]

**Issuer**:

The entity that issues a Verifiable Credential to a Holder.  This could be a bank, government agency or anyone that accepts responsibility for making credentials.  For example a governmental agency can issue a passport to a citizen, or a gym can issue a membership to a member.  The type of agency is not important, only that they issue a credential. This role accepts responsibility for having issued that credential and should look to establish trust and reputation with Holders and Verifiers.  The Issuer's DID will be listed in the Issuer (iss:) field of a Verifiable Credential and can be inspected by anyone wishing to know the origin of the credential.

**Holder**:

A Holder is simply any person or wallet that holds a Verifiable Credential.  Verifiable Credentials can represent anything, a gym membership, an accomplishment like a University degree, or a permission set, such as a login or authentication role.

**Verifier**:

A Verifier is any person or wallet that performs a verification on a Verifiable Credential or any of it's referenced entities.  A Verifier might perform a check on cryptographic elements of a VC, or make sure that the Issuer DID belongs to the expected entity.  Verifiers usually only care to double check that the Verifiable Credential is legitimate and that the included claims meet their expectations, for example when a bartender checks the age of a patron before serving them alcohol.

**Trust Registry**:

When Verifiers need to know who a DID belongs to, there needs to be a way to look up that information.  A Trust Registry is a mapping between DIDs and the entities they represent. You can think of this like a phone book for DIDs.  Trust Registries need to be trusted themselves, and can be as broad or as specific as they need to be.  For example, an Real Estate specific Trust Registry that lists real estate agents can be a way to validate that a particular agent is an accepted member of that industry.
