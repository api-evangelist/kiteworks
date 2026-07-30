---
name: Upload a file to a Kiteworks folder and share it by secure mail
description: Navigate the Kiteworks folder tree, upload a file into a folder, confirm the upload, then send it to external recipients as a tracked secure-email package.
api: openapi/kiteworks-core-openapi-original.json
operations:
  - GET /rest/folders/top
  - GET /rest/folders/{id}/folders
  - POST /rest/folders/{id}/actions/initiateUpload
  - GET /rest/uploads/config
  - GET /rest/uploads/{id}
  - POST /rest/mail/actions/sendFile
  - GET /rest/mail/{id}/recipients
---

# Upload a file to Kiteworks and share it securely

All paths are relative to the tenant instance host and require
`Authorization: Bearer <access_token>` (see `kiteworks-authenticate.md`).

## 1. Find the destination folder

`GET /rest/folders/top` returns the caller's top-level folders. Walk down with
`GET /rest/folders/{id}/folders` until you reach the target, or create one with
`POST /rest/folders/{id}/folders`.

Both listings are offset-paginated: pass `limit` (default 25) and `offset`, and keep
paging until a page returns fewer rows than `limit`. Sort with
`orderBy=name:asc` — remember the colon must be URL-encoded as `%3A`.

## 2. Check upload configuration and quota

`GET /rest/uploads/config` reports what the instance permits. `GET /rest/users/me/quota`
tells you whether the user has room. A `507` on upload means the quota is exhausted, and a
`413` means the file exceeds the configured size limit — check both before pushing bytes.

## 3. Initiate and perform the upload

`POST /rest/folders/{id}/actions/initiateUpload` starts a chunked upload session for the
folder and returns the upload handle. Send the content as multipart MIME. Poll
`GET /rest/uploads/{id}` to confirm the session completed, and `GET /rest/uploads` to list
sessions still in flight. Abandon a bad session with `DELETE /rest/uploads/{id}`.

Uploads are **not** idempotent — Kiteworks publishes no idempotency key. If a POST times
out, list the folder or the upload session to determine whether it landed before retrying,
otherwise you will create a duplicate.

## 4. Send it as a secure mail package

`POST /rest/mail/actions/sendFile` creates the tracked secure-email package carrying the
file to its recipients. Confirm delivery targets with `GET /rest/mail/{id}/recipients`, and
watch `GET /rest/mail/{id}/scanStatus` if content scanning is enabled on the instance.

## 5. Track, report and withdraw

- `GET /rest/mail/{id}/trackings` style reporting is exposed through
  `POST /rest/mail/{id}/actions/sendTrackingReport`.
- `GET /rest/mail/{emailId}/attachments/report` (or `.../reportCsv`) gives the attachment
  access report — this is the audit evidence regulated customers need.
- If the package went out in error, `POST /rest/mail/{id}/actions/withdrawFiles` pulls the
  files back, and `POST /rest/mail/actions/withdraw` withdraws the package.

## Errors

`403` means scope or folder permission, `409` a name conflict, `422` a validation failure
whose response `field` names the offending parameter, `490` a WAF block. Back off on `429`
using `Retry-After`.
