# Unresolved questions

1. What supported, versioned Pulse projection supplies the complete identity inventory from product cards? The current shipped `pulse me` query returns only a self view and cannot be treated as a directory.
2. Which exact source revisions and public verification interfaces define `platform grant/1`, `swarm-envelope/2`, owner-registry attestation, and Keymaker? What valid and refused vectors cover each format?
3. What proves a human's identity, and how does the verified identity-provider `(issuer, subject)` map to exactly one current Pulse-card identity?
4. What agent proof is accepted before per-agent Keyvault keys are available? How will each future key bind to one full MXID, and how are expiry, rotation, and revocation checked?
5. What exact operations and case-sensitive scopes do swarma-lemma and swarma-lease require? What are their current verifier seams, bootstrap transition rules, and negative acceptance checks?
6. Should the first access token be opaque with authenticated RFC 7662 introspection, or a JWT access token using RFC 9068? What issuer, audience, expiry, revocation, cache, and availability rules apply?
7. What service identity authenticates each introspection call, and what transport and failure behavior are supported between the self-hosted services?
8. What are the accepted CLI command grammar, HTTP paths/version, and shipped-skill entry point for sign-in, token use, revocation, and machine-readable errors?
