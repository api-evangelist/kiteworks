# Kiteworks

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

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
