---
name: Provision and deprovision Kiteworks users over SCIM 2.0
description: Use the SCIM 2.0 endpoints to discover the service provider's capabilities and schemas, then create, read, update and deactivate Kiteworks users from an identity provider.
api: openapi/kiteworks-core-openapi-original.json
operations:
  - GET /rest/scim/ServiceProviderConfig
  - GET /rest/scim/Schemas
  - GET /rest/scim/ResourceTypes
  - GET /rest/scim/Users
  - POST /rest/scim/Users
  - GET /rest/scim/Users/{id}
  - PUT /rest/scim/Users/{id}
  - DELETE /rest/scim/Users/{id}
---

# Provision Kiteworks users over SCIM 2.0

Kiteworks exposes a SCIM 2.0 surface so an IdP (Okta, Entra ID, Ping) can own the user
lifecycle instead of the Kiteworks admin console. Fifteen operations in the v28 API carry
the `scim` tag.

## 1. Discover before you provision

Always start with discovery rather than assuming the shape:

- `GET /rest/scim/ServiceProviderConfig` — which SCIM features this instance supports.
- `GET /rest/scim/Schemas` (and `/rest/scim/Schemas/{id}`) — the attribute definitions.
- `GET /rest/scim/ResourceTypes` (and `/rest/scim/ResourceTypes/{id}`) — the resources exposed.

Do not hardcode attributes the instance's schema does not declare.

## 2. Manage users

- `GET /rest/scim/Users` — list and filter users.
- `POST /rest/scim/Users` — create a user.
- `GET /rest/scim/Users/{id}` — read one user.
- `PUT /rest/scim/Users/{id}` — replace the user record.
- `DELETE /rest/scim/Users/{id}` — deprovision.

The API also serves the same resources at the lowercase paths `/rest/scim/users` and
`/rest/scim/users/{id}`. Pick one casing and stay consistent.

## 3. Sequencing rules

Creation is not idempotent and the API publishes no idempotency key. Before `POST`, query
`GET /rest/scim/Users` filtered on the external identifier; a `409` on create means the
user already exists, so reconcile with `PUT` rather than retrying the `POST`.

`422` responses name the offending attribute in the `field` member of the error envelope —
use it to map failures back to the IdP attribute mapping.

## 4. Scope

SCIM calls need a scope covering the `scim` resource family, e.g. `*/scim/*`, whitelisted
on the custom application. A `403` here almost always means the application's scope list is
too narrow, not that the admin lacks rights.
