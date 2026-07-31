---
name: Submit Nift customer deletion requests
description: Submit one or many customer emails for deletion (anonymization) in Nift, safely and idempotently.
api: openapi/nift-partners-openapi.yml
operations: [createCustomerDeletions]
auth: oauth2 client_credentials (scope write:customers)
---

# Submit Nift customer deletion requests

Use this to fulfil a privacy/GDPR erasure request by asking Nift to anonymize a customer.

## Steps

1. **Get an access token.** POST `https://www.gonift.com/oauth/token` (form-encoded) with
   `grant_type=client_credentials`, your `client_id`, `client_secret`, and `scope=write:customers`.
2. **Submit the batch** (`createCustomerDeletions`). POST to
   `https://www.gonift.com/api/v2026-07/partners/customers/deletions` with headers
   `Authorization: Bearer <access_token>` and `Content-Type: application/json`, and a JSON body
   `{ "emails": ["jane@example.com", ...] }` — 1 to 500 addresses per request.
3. **Read the receipt.** A `202 Accepted` returns `received_count` and `queued_count`. Deletion
   runs asynchronously; `queued_count` reflects only de-duplication/validation of your own
   submission and never reveals whether an email belongs to a Nift customer.

## Rules

- **Idempotent:** re-submitting an already-queued email is skipped, so retrying after a network
  failure is safe. Emails are normalized (trimmed + lowercased) before matching.
- Max 500 emails per request; split larger lists.
- `400 invalid_request` = bad `emails` array; `401 invalid_token` = refresh the token;
  `403 insufficient_scope` = token lacks `write:customers`; `403 integration_not_configured` =
  contact Nift to enable deletions.
- Server-side only.
