---
doc_id: partner-ledger-integration-spec
title: Ledger Partner Integration Spec – Corvid ERP Connector
folder: eng-partner
sensitivity: internal
---

# Ledger Partner Integration Spec: Corvid ERP Connector

**Status:** Approved, in development
**Built by:** Northbridge Consulting with the Acme Ledger team
**Target release:** February 2027

## Goal

Let Acme Ledger customers sync invoices and payments into the **Corvid ERP** accounting system automatically, so their finance teams don't re-enter data.

## Scope

In scope:
- Push **finalised invoices** from Ledger to Corvid as receivables.
- Push **payments** and **refunds** to Corvid and mark invoices paid.
- Map Ledger customers to Corvid customer records.

Out of scope (v1):
- Syncing from Corvid back to Ledger.
- Multi-currency (depends on Ledger multi-currency, Q1 2027).

## Architecture

1. The connector runs inside `ledger-connectors` as a separate worker.
2. It subscribes to Ledger webhook events:
   - `invoice.finalized`
   - `payment.succeeded`
   - `refund.created`
3. For each event, it calls the Corvid REST API using the customer's OAuth token, stored encrypted in Ledger.
4. Each sync writes a record to `connector_sync_log` with the Ledger ID, Corvid ID, status and error.

## Mapping

| Ledger | Corvid |
|---|---|
| `customer.id` (`cus_...`) | `Customer.ExternalRef` |
| `invoice.number` | `Receivable.DocNumber` |
| `invoice.total.amount` (minor units) | `Receivable.Amount` (decimal, converted with currency exponent) |
| `invoice.due_date` | `Receivable.DueDate` |
| `payment.amount` | `Payment.Amount` |

## Reliability

- Idempotency: the connector sends the Ledger event ID as the Corvid `Idempotency-Token`.
- Retries: exponential backoff up to **24 hours**, then the sync is marked failed and the customer sees it in the Ledger dashboard.
- Rate limit: Corvid allows 10 requests per second per customer; the worker uses a token bucket per customer.

## Testing

- All development uses the Corvid sandbox and synthetic Ledger tenants in `acme-dev`.
- Contract tests run in CI against a recorded Corvid sandbox.

## Acceptance criteria

- 99.5% of invoices sync within 5 minutes of finalisation in the sandbox load test (10,000 invoices/hour).
- No duplicates under retry.
- Every failed sync is visible to the customer with a readable error.
