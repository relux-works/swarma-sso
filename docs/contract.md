# Swarma SSO service contract (proposal)

Status: proposed for agreement. This document is not a production specification. Interface names marked “candidate” and unresolved items require agreement by the identity and service owners before implementation.

## Scope

The first prototype covers identity authentication and scoped access for swarma-lemma and swarma-lease. Swarma-board retains its current authentication. The SSO consumes a read-only Pulse identity projection; it does not own an independent user list. Existing Swarma trust formats remain authoritative for identity and grant proof.

## Trust and identity

1. Resolve the caller to exactly one full Matrix user ID from a complete, accepted Pulse projection. Include the declared role and superior only as source data; they do not grant service access by themselves.
2. Authenticate that identity using the existing applicable trust path: `platform grant/1`, `swarm-envelope/2`, owner-registry attestation, and Keymaker, as their source contracts specify. Preserve their versions, signatures, audiences, expiry, revocation, and verification rules exactly. Do not encode or reinterpret them as a new grant format.
3. The caller's MXID is an index into the accepted identity projection. A supplied MXID, display name, card role, or unverified environment value alone must never authenticate a request.
4. Reject a missing, partial, ambiguous, stale, or conflicting identity projection. The current Pulse CLI has no complete-directory view; the supported inventory interface and freshness policy are open dependencies.
5. Human proof and the mapping from an accepted human identity-provider subject to one Pulse identity are open. Agent proof before and after Keyvault per-agent key availability is also open. No login path may be declared ready until these proofs are specified.

## Candidate API and versions

Use OAuth 2.0 and OpenID Connect Core 1.0 protocol behavior for login and issuance. Candidate endpoints are:

| Operation | Candidate interface | Result |
| --- | --- | --- |
| Human authorization | OIDC Authorization Code Flow with PKCE | Validated identity-provider response, then one Pulse identity mapping |
| Agent authentication | Existing accepted key/trust proof; exact endpoint and assertion profile pending | Authenticated agent identity, never an MXID-only claim |
| Token issuance | OAuth token endpoint | One service audience and an allowlisted set of that service's scopes |
| Resource verification | RFC 7662 token introspection | `active`, subject, audience, scopes, issuer and expiry needed for the caller's decision |

The service-resource adapter is proposed as `sso-resource/v1`. This name and endpoint paths are not agreed. The token format proposal is an opaque random bearer value carried as `Authorization: Bearer <token>`; it is not a serialized Swarma grant. Do not create a `grant/3`, new envelope, or private claim convention. If JWT is selected instead, use RFC 9068 and separately agree signing-key discovery, rotation, audience, and revocation.

For each token, `aud` is exactly one registered resource identifier. Each scope is an exact, case-sensitive service action. Candidate names follow `<service-id>:<action>`; actual action names and whether the delimiter is retained are pending the service owners. No wildcard, unknown, or cross-service scope is accepted. The SSO grants only the intersection of requested scopes, authenticated identity policy, and that service's allowlist.

Candidate introspection request: authenticated protected-resource client, HTTPS POST, form field `token`, plus the target service context if needed. Candidate active response fields: `active: true`, exact `sub`, singleton `aud`, exact space-delimited `scope`, `iss`, and `exp`. Inactive or invalid tokens return `active: false`. The service checks the expected issuer, its own audience, expiry, active state, and every scope required for the operation before calling its protected handler. The exact wire version, service authentication, transport, response schema, and cache policy remain to be agreed.

## Verification seam

Each service keeps one middleware boundary around protected operations. A candidate verifier accepts `(request, service_id, required_scopes)` and returns an authenticated principal only after token introspection succeeds. It denies by default when SSO is unavailable, the token is inactive, the issuer or audience is wrong, or any required scope is absent. It never falls back to a bootstrap token after SSO mode is enabled. Before SSO cutover, each service retains its verified current authentication through a service-owned seam agreed with that service's owner; the existing mechanism and migration baseline remain to be established.

The service passes its own audience and required action scopes. The SSO never selects a service from the token alone, and a valid token for one service cannot authorize another. Authorization must complete before the handler can perform a write or other protected effect.

## Errors and refusals

| Condition | Candidate response | Required behavior |
| --- | --- | --- |
| No authentication information | HTTP 401 challenge without detailed error | Do not invoke the handler |
| Malformed bearer request | HTTP 400 with `invalid_request` | Do not echo the supplied token |
| Unknown, expired, revoked, malformed, or wrong-audience token | HTTP 401 with `invalid_token` | Do not invoke the handler |
| Valid token missing a required scope | HTTP 403 with `insufficient_scope` | Do not invoke the handler; disclose only the required scope if approved |
| SSO/introspection unavailable or invalid response | HTTP 503, fail closed | Do not treat the token as active or use static-auth fallback |
| Identity proof invalid or ambiguous | Generic login refusal | Do not reveal whether a particular MXID or key is registered |

Bearer error names and recommended status mappings follow RFC 6750. The 503 behavior, response body schema, retry policy, and information exposed to clients require service-owner agreement.

## Acceptance checks by side

### SSO identity and authentication

- Positive: one complete pinned Pulse projection plus one valid, current accepted proof maps to exactly one full MXID and preserves source role/superior values.
- Negative: unknown MXID, duplicate mapping, missing/partial projection, wrong key binding, bad signature, expired or revoked proof, malformed trust record, or missing human identity mapping refuses without creating a local identity or issuing a token.
- Positive: a request for one allowed scope and one audience issues a token bounded to that audience and scope.
- Negative: unknown audience, unknown scope, wildcard, cross-service scope, unauthenticated caller, or scope outside the identity's accepted grant refuses with no token issued.

### Each service adapter (swarma-lemma and swarma-lease)

- Positive: a current token for that service with its required exact scope reaches the handler.
- Negative: missing, malformed, expired, revoked, wrong-issuer, wrong-audience, inactive, or insufficient-scope token is refused before the handler; the neighboring valid token for that same service still succeeds.
- Negative: introspection timeout, malformed response, or unverifiable SSO result fails closed and does not use the static bootstrap credential.
- Pre-cutover positive: each service's source-verified current authentication continues to work. After accepted SSO cutover, the former authentication path cannot bypass SSO.
- Denied requests have no protected side effect and do not log or return token bytes.

### Clients

- Human public clients use transaction-specific PKCE; login state and tokens do not appear in command output, skill text, URLs, or logs.
- Agent clients authenticate using only the accepted per-agent trust proof; no client can choose another MXID by changing a request field.
- CLI, HTTP API, and shipped skill use the same agreed scope and refusal semantics. Their exact command grammar and API paths are not yet specified.

### Swarma-board

- Positive: its current authentication path continues to work during the first prototype.
- Negative: enabling the first SSO integrations does not silently migrate, weaken, or bypass board authentication.

## Open contract items

See [UNRESOLVED_QUESTIONS.md](../UNRESOLVED_QUESTIONS.md). Implementation is gated on the complete Pulse inventory interface, authoritative trust-format sources and vectors, human and agent proof mapping, exact per-service scopes, and both service owners' acceptance of the verifier contract.
