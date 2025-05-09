# Verification {#sec-verification}

## Presenting proof

Verification is a crucial interaction between Self-Sovereign Identity (SSI) agents. This process involves a `verifier` creating a present proof request to a `holder`, who then responds with a proof presentation. The verifier agent subsequently verifies this presentation.

The verification process consists of several key steps:

1. The verifier initiates a present proof request.
2. The holder responds with a proof presentation.
3. The verifier agent automatically verifies the presentation's cryptographic integrity.
4. The verifier makes a final decision to accept or reject the Verifiable Credential (VC).

It's important to note that a valid and correctly formatted VC alone is insufficient for acceptance. While the verifier agent automatically checks the cryptography, the verifier must also:

1. Confirm the trustworthiness of the issuer.
2. Evaluate and accept the claims within the VC.

Thus, the verifier has the final say in accepting or rejecting the verified proof presentation.

The core of the verification process involves the holder signing a random challenge along with the presentation. The verifier agent extracts the public key from this signature and compares it with the public key of the VC's `subject`. If these public keys match, it confirms that the entity responding to the proof presentation request has access to the private key of the VC's subject.

## Presentation policies

To streamline the verification process, verifier cloud agents can implement `presentation policies`. These policies serve as an automated mechanism to enhance efficiency and consistency in verifying Verifiable Credentials (VCs).

Key aspects of presentation policies include:

1. Trusted Issuer Lists: 
   - Policies establish sets of `trusted issuers` for specific VC `schemas`.
   - This allows verifiers to pre-approve credible sources for particular types of credentials.

2. Dynamic Management:
   - Policies can be updated and managed over time.
   - Verifiers can add or remove trusted issuers as needed, adapting to changes in the ecosystem or their own requirements.

3. Automated Rejection:
   - The verifier cloud agent can automatically reject a presentation proof if the issuer is not listed in the relevant presentation policy.
   - This reduces manual intervention and speeds up the verification process for non-compliant credentials.

4. Schema-Specific Trust:
   - Policies are tied to specific VC schemas, allowing for granular control over which issuers are trusted for different types of credentials.

By implementing presentation policies, verifiers can significantly reduce the manual effort required in the verification process, while maintaining control over which issuers they deem trustworthy for different types of credentials. This approach balances automation with the flexibility to adapt to changing trust relationships in the Self-Sovereign Identity ecosystem.

## Selective disclosure

Standard Verifiable Credentials (VCs) presented in JWT format require holders to disclose all content for verifiers to validate integrity and confirm authenticity. However, this approach can compromise privacy when verifiers only need to check specific claims.

Consider a common scenario: age verification at a bar. While establishments typically only need to confirm if a customer is of legal drinking age (18 or 21 in many jurisdictions), traditional ID checks reveal excessive personal information, including full name, address, and exact date of birth. This over-disclosure becomes particularly problematic in digital interactions, where shared information can be easily copied and archived across the internet.

To address this privacy concern, the concept of `selective disclosure` was developed. This approach utilizes `zero knowledge proofs`, a cryptographic technique allowing the sharing of specific information subsets without revealing the entire content.

Hyperledger Identus currently supports two VC formats that enable selective disclosure proofs:

1. SD-JWT (Selective Disclosure JSON Web Token)
2. AnonCreds (Anonymous Credentials)

These formats allow holders to prove specific claims (e.g., being over 21) without disclosing unnecessary personal details, striking a balance between verification needs and privacy protection.

Some of the benefits of Selective Disclosure include:

1. Enhanced Privacy: Minimizes the exposure of sensitive personal information.
2. Compliance: Aligns with data protection regulations by adhering to data minimization principles.
3. User Control: Empowers individuals to manage their digital identity more effectively.
4. Reduced Risk: Limits the potential for data breaches or misuse of personal information.

TODO: Add API request and code examples (Milestone 3)
