---
doc_id: eng-data-retention-impl
title: Data Retention Implementation
folder: eng
sensitivity: internal
---

# Data Retention Implementation

How Beacon and Ledger actually delete data. The legal retention commitments are set by Legal; this page explains how engineering implements them.

## Beacon hot data (ClickHouse)

| Plan | Hot retention |
|---|---|
| Starter | 7 days |
| Team | 30 days |
| Enterprise | 90 days (365 days optional add-on) |

- ClickHouse tables are partitioned **by day and tenant retention tier**.
- A `TTL` clause on each table drops partitions older than the tier's retention.
- The `retention-auditor` job runs daily at 04:00 UTC and alerts if any partition is older than its retention + 1 day.

## Beacon exports (S3)

- Export files expire after **7 days** via an S3 lifecycle rule.
- Since INC-2041, export object keys include a random 128-bit component.

## Beacon archive (Enterprise extended retention)

- Stored in S3, transitioned to Glacier after 30 days.
- Deleted at the end of the contracted archive period (1–7 years).

## Ledger

- Invoices and payment records are kept for **7 years** to meet financial record-keeping rules. These can't be deleted on customer request; they are anonymised instead where possible.
- Raw usage events are deleted **13 months** after the billing period closes.

## Customer deletion requests

1. Support opens a `DEL-xxxx` ticket after Legal confirms the request.
2. The `tenant-purge` job deletes the tenant's data from ClickHouse, Postgres, S3 and backups' next rotation.
3. Deletion must complete within **30 days** of the request.
4. The job writes a deletion certificate (timestamp, data stores, row counts) to the ticket.

## Backups

- RDS/Aurora snapshots: daily, retained **35 days**.
- ClickHouse backups: daily to S3, retained **14 days**.
- Deleted tenant data disappears from backups when those backups expire. This is documented in the customer DPA.

## Employee and internal data

Internal logs in the `acme-internal` Beacon tenant are retained for 90 days. Access logs for production systems (including S3 server access logs) are retained for **1 year** for security investigations.
