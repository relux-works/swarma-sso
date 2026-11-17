# Swarma SSO

Swarma SSO is intended to provide one sign-in point for human and agent identities and issue narrowly scoped access to Swarma services. The first planned service integrations are swarma-lemma and swarma-lease. Swarma-board keeps its existing authentication during the first prototype.

Identity membership is sourced from the accepted Pulse product-card projection. A full Matrix user ID is a lookup key, not proof of identity. The authentication and authorization design must preserve the existing Swarma trust formats and must not introduce another grant format.

## Current status

This repository is a documentation scaffold. It does not yet contain a runnable SSO service, CLI, HTTP API, or shipped skill. The authentication provider, Pulse directory projection, agent key proof, service scopes, and verifier interfaces still require a recorded contract before implementation.

See [research](docs/research.md), the [proposed contract](docs/contract.md), and [unresolved questions](UNRESOLVED_QUESTIONS.md).

## Tools

There are no product build, test, or run commands in this scaffold. No executable product tool is available yet. When one is added, this section will name its purpose, exact entry point or command, and output location.

## License

Licensed under the Apache License, Version 2.0. See [LICENSE](LICENSE).
