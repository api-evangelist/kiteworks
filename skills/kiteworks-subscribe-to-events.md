---
name: Subscribe to Kiteworks platform events with PubSub webhooks
description: Register a signed HTTPS webhook against the Kiteworks PubSub service, choose NATS subject filters for the events you care about, and verify each delivery's HMAC signature.
api: openapi/kiteworks-pubsub-openapi-original.yml
operations:
  - GET /webhooks
  - POST /webhooks
  - GET /webhooks/{id}
  - PUT /webhooks/{id}
  - PATCH /webhooks/{id}
  - DELETE /webhooks/{id}
---

# Subscribe to Kiteworks events

Kiteworks delivers real-time events through PubSub, a NATS-backed bus fronted by a webhook
registration API at `/pubsub-ext/webhooks` on the tenant instance. There is no AsyncAPI
document — the registration API is described with OpenAPI and the event catalog is
documented prose.

## 1. Register a webhook

`POST /webhooks` with:

- `url` (**required**) — your HTTPS endpoint. Plain HTTP is rejected.
- `secret` (**required**) — the shared secret used to sign deliveries.
- `subscriptions` (**required**) — an array of NATS subject filters.
- `token` (optional) — sent back as an outbound authorization header, so your receiver can
  authenticate Kiteworks in turn.
- `enabled` (optional) — defaults to `true`.

Administrators can do the same through Admin Portal → Application Setup → Event Subscriptions.

## 2. Write the subject filters

Subjects are dot-separated. `*` matches exactly one segment; `>` matches all remaining
segments. Subject characters are limited to letters, numbers, dot, asterisk, underscore and
hyphen, optionally ending in `>`.

Documented subjects include `events.user.login`, `events.user.logout`,
`events.filesystem.upload` and `filesystem.file.create`. Subscribe broadly with
`filesystem.>` and filter on your side, or subscribe narrowly to keep volume down.

Kiteworks does not publish an exhaustive event catalog — treat unknown `event_name` values
as forward-compatible and ignore rather than reject them.

## 3. Verify every delivery

Each POST carries an HMAC-SHA256 signature in the **`X-KW-Signature`** header, computed
with the registration secret. Recompute it over the raw body and compare in constant time
**before** parsing. Reject anything that does not match.

The envelope is:

```json
{ "tenantId": "...", "webhookId": "...", "payload": { ... } }
```

`tenantId` is internal — ignore it. `webhookId` identifies which registration fired.
`payload` is the event, with fields such as `event_name`, `user_id`, `username`,
`ip_address`, `created`, `successful` and `guid`.

## 4. Manage the registration

`GET /webhooks` lists registrations (offset-paginated), `GET /webhooks/{id}` reads one,
`PUT /webhooks/{id}` replaces it, `PATCH /webhooks/{id}` partially updates it — use PATCH
to flip `enabled` during an incident — and `DELETE /webhooks/{id}` removes it.

Rotate the secret by PATCHing a new `secret`, and accept both the old and new signature for
the length of the rollover.
