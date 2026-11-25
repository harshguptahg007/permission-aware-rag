---
doc_id: eng-api-style-guide
title: API Style Guide
folder: eng
sensitivity: internal
---

# API Style Guide

How Acme designs public REST APIs (Beacon API v2, Ledger API). This guide is shared with integration partners working on Ledger connectors.

## URLs and versioning

- Base path includes the major version: `/v2/...` (Beacon), `/v1/...` (Ledger).
- Resources are plural nouns in kebab-case: `/v1/usage-events`, `/v1/invoices`.
- Breaking changes require a new major version. Non-breaking additions (new optional fields, new endpoints) don't.
- A deprecated version is supported for at least **12 months** after its replacement is GA.

## Methods

| Action | Method | Example |
|---|---|---|
| List | GET | `GET /v1/invoices` |
| Read | GET | `GET /v1/invoices/{id}` |
| Create | POST | `POST /v1/invoices` |
| Partial update | PATCH | `PATCH /v1/customers/{id}` |
| Delete | DELETE | `DELETE /v1/webhooks/{id}` |
| Action | POST to sub-resource | `POST /v1/invoices/{id}/finalize` |

## Request and response bodies

- JSON, `snake_case` field names.
- Timestamps in ISO 8601 UTC: `2026-11-03T14:05:00Z`.
- Money: `{ "amount": 12500, "currency": "USD" }` (integer minor units).
- IDs are opaque strings with a type prefix: `inv_...`, `cus_...`, `evt_...`.

## Pagination

- **Cursor-based** for all list endpoints: `?limit=50&starting_after=<id>`.
- Default limit 50, maximum 200.
- Responses include `has_more` and `next_cursor`.
- Offset pagination is not allowed in new APIs.

## Idempotency

- Every `POST` that creates or changes money-related resources **must** accept an `Idempotency-Key` header.
- Keys are stored for **24 hours**. A repeated key with the same body returns the original response; with a different body, it returns `409 Conflict`.

## Errors

```json
{
  "error": {
    "type": "invalid_request",
    "code": "missing_field",
    "message": "customer_id is required",
    "param": "customer_id",
    "request_id": "req_8f2a..."
  }
}
```

| Status | When |
|---|---|
| 400 | Invalid input |
| 401 | Missing or invalid API key |
| 403 | Authenticated but not allowed |
| 404 | Not found (also used when the resource belongs to another tenant) |
| 409 | Idempotency conflict or state conflict |
| 429 | Rate limit exceeded; includes `Retry-After` |
| 5xx | Our fault; safe to retry with backoff |

## Rate limits

Default **100 requests per second per API key**, with burst to 200. Enterprise plans can be raised. Headers: `X-RateLimit-Limit`, `X-RateLimit-Remaining`.

## Webhooks

- Signed with HMAC-SHA256 in the `Acme-Signature` header.
- Retried with exponential backoff for up to **3 days**.
- Receivers must respond `2xx` within **10 seconds**.
