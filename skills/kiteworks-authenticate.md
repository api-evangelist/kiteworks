---
name: Authenticate to the Kiteworks REST API
description: Obtain a Kiteworks OAuth 2.0 access token using either the authorization code flow with PKCE (interactive users) or the JWT bearer grant (unattended server jobs), then call an authenticated endpoint.
api: openapi/kiteworks-core-openapi-original.json
operations:
  - GET /rest/users/me
  - GET /rest/users/me/quota
---

# Authenticate to the Kiteworks REST API

Kiteworks is a per-tenant appliance. Every URL below is relative to the customer's own
instance host, e.g. `https://files.example.com`. There is no shared multi-tenant API host,
and there are no API keys — OAuth 2.0 is the only supported scheme.

## Before you start

- The instance must run the Enterprise package with the **Developer Suite** add-on enabled
  (Admin console → Application Setup → Licenses; the API row must read "Enabled").
- A **custom application** must be registered on the instance, giving you `client_id`,
  `client_secret`, the permitted scopes, redirect URIs and token lifetimes.

## Choose a flow

**Authorization code + PKCE** — use for anything a human signs into (web, desktop, mobile).
PKCE is required for public clients.

1. Send the user to `https://{instance}/oauth/authorize` with `client_id`, `redirect_uri`,
   `scope`, `response_type=code`, a `state` value, and the PKCE `code_challenge`.
2. Exchange the returned `code` at `https://{instance}/oauth/token` with `grant_type=authorization_code`
   and the `code_verifier`.
3. The response carries `access_token`, `expires_in`, `token_type` (always `bearer`), `scope`,
   and a `refresh_token` when the application is configured to issue one.

**JWT bearer** — use for scheduled jobs and unattended scripts. No refresh token is issued;
mint a fresh assertion each time.

1. Build a JWT with claims `iss`, `sub` (the Kiteworks user's email), `aud`, `iat`, `nbf`,
   `exp` (keep it short — around 5 minutes), and a unique `jti` to block replay.
2. Sign it with the RSA private key whose public half is registered in the Admin console (RS256).
3. POST it to `https://{instance}/oauth/token` with
   `grant_type=urn:ietf:params:oauth:grant-type:jwt-bearer`.

## Scopes

Scopes are not a fixed enumerated list. They are pattern-based —
`{METHOD}/{resource}/{qualifier}` with `*` as a wildcard — and are whitelisted per
application in the Admin console. Request them as a space-separated string. Documented
examples are `GET/users/*` and `*/files/*`. The resource segment matches the API's tag
names (`files`, `folders`, `mail`, `users`, `scim`, `search`, …).

## Verify the token

Call `GET /rest/users/me` with `Authorization: Bearer <access_token>`. A 200 returns the
current user. Use `GET /rest/users/me/quota` to confirm storage headroom before uploading.

## Failure handling

- `401` — the token is missing, expired or invalid. Refresh or re-mint, then retry once.
- `403` — the user is allowed but the granted **scope** is not. Widen the scope in the
  Admin console; retrying will not help.
- `490` — the request was blocked by the Kiteworks WAF, not by your credentials.
- `429` — back off exponentially starting at 1 second, honouring `Retry-After`.

There is no idempotency-key contract on this API, so never blindly retry a non-GET request
that may have partially succeeded.
