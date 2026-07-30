# Kiteworks

Kiteworks operates a Private Data Network (PDN) that unifies secure email, secure file sharing, secure web forms, managed file transfer (MFT), and SFTP behind a single hardened virtual appliance with a common governance, encryption, and audit layer. It serves regulated industries — government, defense, healthcare, financial services, legal, manufacturing, and higher education — where sensitive content leaving the organization must be tracked and provably compliant. The company was formerly known as Accellion.

- Website: https://www.kiteworks.com/
- Developer portal: https://developer.kiteworks.com/
- GitHub: https://github.com/kiteworks

## APIs

| API | Spec | Notes |
|---|---|---|
| Kiteworks Core API | `openapi/kiteworks-core-openapi-original.json` | OpenAPI 3.0.2, version 28 — 304 paths / 394 operations across 40 tags (files, folders, mail, users, admin, SCIM, search) |
| Kiteworks PubSub Consumer API | `openapi/kiteworks-pubsub-openapi-original.yml` | OpenAPI 3.0.3, version 1.0.0 — webhook registration for the NATS-backed event bus |

Kiteworks is a per-tenant appliance, so base URLs are templated on the customer instance
host (`https://{instance}/rest`) rather than a shared multi-tenant API host.

## Artifacts

- `authentication/`, `scopes/` — OAuth 2.0 authorization code (PKCE) and JWT bearer; pattern-based scopes (`{METHOD}/{resource}/{qualifier}`)
- `conventions/` — offset pagination, `orderBy` sorting, proprietary `{code, message, field}` error envelope; **no idempotency contract**
- `errors/`, `data-model/` — derived from the v28 OpenAPI (including the non-standard `490` WAF status)
- `rate-limits/` — 30 req/s standard, 300 req/s MFT, `429` + `Retry-After`
- `asyncapi/` — PubSub webhook catalog; HMAC-SHA256 signing via `X-KW-Signature` (no AsyncAPI document is published)
- `mcp/` — the official Apache-2.0 Kiteworks MCP Server (20 tools, OAuth 2.1, FIPS 140-3 mode)
- `skills/` — five generated Agent Skills grounded in verified spec paths
- `security/`, `conformance/`, `well-known/` — PGP-signed security.txt, UpGuard trust center, and the published compliance framework list
