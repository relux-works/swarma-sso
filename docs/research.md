# Swarma SSO research

Date: 2026-11-17

Status: research findings and design options; not an agreed service contract.

## Product constraints

- Human and agent identities come from Pulse product cards. The SSO must not keep a separate user directory.
- The exact full Matrix user ID is a lookup key only. It does not authenticate the caller.
- The design must preserve the existing `platform grant/1`, `swarm-envelope/2`, owner-registry attestation, and Keymaker contracts. It must not define a third grant format.
- The first proposed service adapters are swarma-lemma and swarma-lease. Their current authentication mechanisms and supported verifier seams require source verification before migration. Swarma-board retains its existing authentication for the first prototype.
- Planned client interfaces include a CLI, an HTTP API, and a shipped skill, usable from authorized client machines.

## Identity-source boundary

The installed Pulse query contract reads one explicitly selected `product-card/v3` snapshot and returns a self-only view. Its optional environment and discovery views have different, bounded semantics; discovery is caller-relative and is not a complete identity directory. Therefore the current `pulse me` interface cannot by itself supply the complete card-derived identity set required by SSO.

Preferred option: obtain a supported, versioned Pulse projection for the identity inventory, with exact source pins and the required full IDs and declared role/superior data. SSO should consume that projection read-only and reject partial or ambiguous inventory rather than invent missing members.

Alternative: have SSO parse card files itself. This duplicates Pulse's schema and compatibility rules and risks diverging from the accepted card reader, so it is not recommended unless the Pulse interface owner explicitly establishes that as the supported seam.

## Authentication and token options

### Human sign-in

For public human clients, evaluate OpenID Connect Authorization Code Flow with PKCE. Validate the configured issuer, audience, nonce, signature, and protocol response before mapping the verified `(issuer, subject)` to exactly one accepted Pulse identity. The identity provider and mapping rules are not specified yet.

### Agent sign-in

Future per-agent Keyvault key pairs could support a signed client assertion, but only if the existing Keyvault, owner-registry, and Keymaker contracts authorize and define that proof. RFC 7523 separates client authentication from the authorization grant; that separation fits the requirement to authenticate a key without creating a new grant format. No agent may be identified by a caller-supplied MXID alone. The transition proof before per-agent keys are available remains open.

### Service access tokens

Two candidates fit OAuth resource-server use:

| Candidate | Benefit | Cost |
| --- | --- | --- |
| Opaque random bearer token plus RFC 7662 introspection | No new signed grant envelope or public claim layout; central active, expiry, revocation, audience, and scope checks | Each service depends on protected SSO introspection; latency and outage behavior must be agreed |
| JWT access token using RFC 9068 | A service can verify claims locally and avoid a request to SSO | Requires signing-key publication, rotation and revocation rules, strict audience validation, and an agreed JWT profile |

Prototype recommendation: use a high-entropy opaque token and authenticated RFC 7662 introspection. Store only a verifier (for example, a cryptographic digest) for each token at rest. Introspection must return inactive for unknown, expired, revoked, or wrong-audience tokens. This is a proposal, not an agreed token representation or storage contract.

### Scopes and service audiences

Keep the token audience separate from its scopes: one token targets one service, and its scopes name only actions that service has approved. OAuth scope values are case-sensitive and their meaning is authorization-server-defined. A service-qualified exact name such as `swarma-lemma:<service-owned-action>` is a candidate vocabulary; actual actions must come from each service's API and authorization rules. Do not issue wildcard scopes or combine unrelated service audiences. RFC 8693 describes why combining several audiences and scopes can unintentionally request their Cartesian product; RFC 9068 also requires JWT scope values to make sense for the token audience when that format is selected.

## Standards findings

1. OAuth access tokens represent authorization with specific scopes and durations, enforced by the authorization server and resource server; they let the resource server consume a token without implementing each underlying authentication method. A scope's meaning is defined by the authorization server and scope values are case-sensitive. [RFC 6749 §1.4](https://www.rfc-editor.org/rfc/rfc6749.html#section-1.4) and [§3.3](https://www.rfc-editor.org/rfc/rfc6749.html#section-3.3).
2. Resource/audience identifies the target service, while scope identifies rights. Avoid multi-service token requests in the first prototype. [RFC 8693 §2.1.1](https://www.rfc-editor.org/rfc/rfc8693.html#section-2.1.1)
3. RFC 7662 defines an authorization-server endpoint for a resource server to ask whether a token is active and learn its authorization metadata, including scopes. [RFC 7662](https://www.rfc-editor.org/rfc/rfc7662.html)
4. If JWT access tokens are selected, use the registered JWT access-token profile rather than an invented claim layout, and bind scope meaning to the token audience. [RFC 9068 §2.2.3](https://www.rfc-editor.org/rfc/rfc9068.html#section-2.2.3)
5. OAuth security guidance calls for minimum privilege and audience-restricted tokens. Public clients use Authorization Code Flow with PKCE. [RFC 9700 §§2.1.1, 2.3](https://www.rfc-editor.org/rfc/rfc9700.html#section-2.3)
6. OpenID Connect Authorization Code Flow returns tokens from the token endpoint; a client validates the ID Token before relying on its subject. [OpenID Connect Core §3.1](https://openid.net/specs/openid-connect-core-1_0-final.html#CodeFlowAuth)
7. JWT client authentication is distinct from a JWT authorization grant. RFC 10070 updates the client-authentication audience rule to the authorization-server issuer identifier as the sole audience. [RFC 7523](https://www.rfc-editor.org/rfc/rfc7523.html), [RFC 10070 §4](https://www.rfc-editor.org/rfc/rfc10070.html#section-4)
8. For bearer-token resource errors, RFC 6750 defines `invalid_token` with 401 and `insufficient_scope` with 403 as the recommended responses. [RFC 6750 §3.1](https://www.rfc-editor.org/rfc/rfc6750.html#section-3.1)

## Required internal evidence

The field-level and verification contracts for the named Swarma trust formats are not included in this repository. Before implementation, the accepted sources and positive/negative vectors must establish how SSO validates each format, binds public keys to an identity, handles expiry/revocation, and rejects malformed or ambiguous evidence. Public standards describe OAuth/OIDC behavior but do not define those internal trust records.
