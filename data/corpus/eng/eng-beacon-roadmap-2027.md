---
doc_id: eng-beacon-roadmap-2027
title: Acme Beacon Engineering Roadmap 2027
folder: eng
sensitivity: internal
---

# Acme Beacon Engineering Roadmap 2027

**Owner:** Beacon Product engineering
**Status:** Draft, under review with Product Management. Dates are targets, not commitments.

## Themes

1. **Open standards:** first-class OpenTelemetry.
2. **Faster answers:** help customers find the cause of an incident faster.
3. **Trust:** data residency, security and reliability, especially after INC-2041.
4. **Cost to serve:** reduce infrastructure cost per GB ingested.

## Quarterly plan

| Quarter | Item | Notes |
|---|---|---|
| Q1 2027 | **Native OTLP ingest** | Accept OpenTelemetry protocol directly in `beacon-ingest`, without the Beacon agent |
| Q1 2027 | Retire Beacon API v1 (stage 1) | Read-only mode for v1 write endpoints; see Legacy API Migration Guide |
| Q2 2027 | **AI log summaries** | Summarise error spikes in plain language in the alert notification; runs inside our AWS account |
| Q2 2027 | ClickHouse upgrade to managed service evaluation | Decide self-managed vs managed by end of Q2 |
| Q3 2027 | **EU data residency in Frankfurt** (`eu-central-1`) | In addition to London; demand from EU healthcare prospects |
| Q3 2027 | Beacon API v1 shut down | 30 June 2027 is the announced deprecation date |
| Q4 2027 | Tiered storage for long retention | Move cold data to S3 + Parquet, query on demand |

## Cost targets

Reduce infrastructure cost per GB ingested by **25%** by end of 2027, mainly through ClickHouse compression tuning and tiered storage.

## Security and trust items

- Continuous cloud posture scanning (from INC-2041, AI-8).
- SOC 2 Type II renewal (Q2).
- Customer-managed encryption keys for Enterprise (Q4, stretch).

## Out of scope for 2027

- A mobile app.
- On-premise Beacon.

## Pricing and packaging

Any changes to plans, pricing or packaging are owned by leadership and Product Management and are not part of this engineering roadmap. Engineering will be told about changes that need product work (for example, new plan limits) at least one quarter ahead.

## Staffing

The roadmap assumes the 2027 engineering headcount plan. If headcount changes, items will be re-prioritised by the VP Engineering, starting from the bottom of this list.
