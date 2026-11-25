---
doc_id: exec-beacon-pricing-change
title: Beacon Pricing Change 2027 – Per-GB Ingest Pricing
folder: exec
sensitivity: restricted
---

# Beacon Pricing Change 2027: Per-GB Ingest Pricing

**Leadership only until announced to customers on 1 February 2027.**

## Decision

From **1 April 2027**, Acme Beacon moves from **per-host pricing** to **per-GB ingest pricing** for all new customers. Existing customers move at their first renewal after 1 April 2027.

## Why

- Per-host pricing undercharges customers with high log volume per host (containers, serverless) and overcharges small hosts.
- Competitors have moved to ingest-based pricing.
- Ingest-based pricing lines up revenue with our largest cost (ClickHouse storage and compute).

## New price list (list prices, annual)

| Plan | Current (per host / month) | New (per GB ingested / month) | Included retention |
|---|---|---|---|
| Starter | $15 | $0.50 per GB | 7 days |
| Team | $31 | $0.90 per GB (+15% effective for the median customer) | 30 days |
| Enterprise | $55 | Custom, from $1.20 per GB | 90 days |

- Committed-volume discounts: 10% at 10 TB/month, 20% at 50 TB/month.
- Extended retention (365 days): +40%.

## Expected impact

- Existing customers: median effective price change **+8%**; about 25% of customers see a decrease.
- Net effect: **+6% ARR** from the installed base over 12 months after renewals.
- Churn risk: modelled at +1 percentage point of gross logo churn in 2027.

## Grandfathering

- Customers with renewals before 1 April 2027 keep per-host pricing for one more term.
- Customers affected by INC-2041 (37 accounts) keep their current pricing for **12 months** after their next renewal, as a goodwill measure.
- Any customer whose price increases by more than 25% gets a 2-year phase-in.

## Rollout

| Date | Step |
|---|---|
| Dec 2026 | Finalise with Sales and Finance |
| Jan 2027 | Billing changes in Ledger (Acme uses Ledger for Beacon billing) |
| 1 Feb 2027 | Customer announcement; sales enablement |
| 1 Apr 2027 | Effective for new customers |

## Confidentiality

Do not share with customers, partners or non-leadership staff before 1 February 2027. Engineering will be told about the Ledger billing work in December with the minimum detail needed.
