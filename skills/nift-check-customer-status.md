---
name: Check Nift customer status
description: Look up whether a customer is eligible to receive a Nift gift and when they last activated or selected one.
api: openapi/nift-partners-openapi.yml
operations: [getCustomerStatus]
auth: oauth2 client_credentials (scope read:customers)
---

# Check Nift customer status

Use this to find out a customer's Nift eligibility before offering them a gift.

## Steps

1. **Get an access token.** POST `https://www.gonift.com/oauth/token` (form-encoded) with
   `grant_type=client_credentials`, your `client_id`, `client_secret`, and `scope=read:customers`.
   The response returns a long-lived (~1 year) `access_token`. Store it server-side and only
   request a new one when it expires. Never expose the client secret or token in client code.
2. **Look up the customer** (`getCustomerStatus`). POST to
   `https://www.gonift.com/api/v:2023-03/partners/customers/status` with header
   `Authorization: Bearer <access_token>` and form fields `email` (customer email) and
   `code` (your partner referral code).
3. **Read the result.** The JSON response has `status` (`available` = eligible for a gift,
   `selected` = already chose one), plus `last_activated_at` and `last_selected_at` timestamps.

## Rules

- Auth failures return an empty body with the error in the `WWW-Authenticate` header
  (`error="invalid_token"`); refresh the token and retry.
- Call this from your server only.
