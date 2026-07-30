---
name: Request files from an external party and collect the uploads
description: Use Kiteworks Request Files to invite someone outside the organization to upload into a controlled location, then enumerate, preview and download what they submitted.
api: openapi/kiteworks-core-openapi-original.json
operations:
  - POST /rest/folders/{id}/actions/requestFile
  - POST /rest/mail/actions/requestFile
  - GET /rest/requestFile/{ref}
  - GET /rest/requestFile/{ref}/uploads
  - GET /rest/requestFile/{ref}/sources
  - GET /rest/requestFile/{ref}/sources/{object_id}/content
  - POST /rest/requestFile/{ref}/reply
  - DELETE /rest/requestFile/{ref}
---

# Request files from an external party

Request Files is the inbound half of Kiteworks: instead of sending content out, you invite
an external counterparty to deposit content into a location you control, with the same
audit trail. This is the pattern behind CMMC / HIPAA style intake workflows.

## 1. Create the request

Either scope it to a folder with
`POST /rest/folders/{id}/actions/requestFile`, or send it as a mail-driven request with
`POST /rest/mail/actions/requestFile`. Both return a request reference (`ref`) that keys
every subsequent call.

## 2. Inspect the request

`GET /rest/requestFile/{ref}` returns the request itself. Kick off an upload session on
behalf of the request with `POST /rest/requestFile/{ref}/actions/initiateUpload`, or accept
a direct submission with `POST /rest/requestFile/{ref}/actions/file`.

## 3. Collect what arrived

- `GET /rest/requestFile/{ref}/uploads` lists upload sessions against the request;
  `GET /rest/requestFile/{ref}/uploads/{object_id}` is one session and
  `.../uploads/{object_id}/content` streams its bytes.
- `GET /rest/requestFile/{ref}/sources` lists the resulting source objects;
  `GET /rest/requestFile/{ref}/sources/{object_id}/content` downloads one.
- `GET /rest/requestFile/{ref}/preview/{object_id}` renders a preview without a full
  download — prefer it when you only need to triage.

All list endpoints are offset-paginated with `limit` / `offset`.

## 4. Correspond and close out

`POST /rest/requestFile/{ref}/comment/{object_id}` annotates a specific submission and
`POST /rest/requestFile/{ref}/reply` responds to the requester. When intake is finished,
`DELETE /rest/requestFile/{ref}` retires the request so the link stops accepting content.

## Notes

Leaving a request open is the common mistake — the link keeps accepting uploads until it is
deleted or expires. Delete it as soon as intake closes. Errors follow the platform envelope
(`code`, `message`, and `field` on 422); `490` is a WAF block rather than an application error.
