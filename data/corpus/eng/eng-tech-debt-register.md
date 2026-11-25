---
doc_id: eng-tech-debt-register
title: Tech Debt Register
folder: eng
sensitivity: internal
---

# Tech Debt Register

Reviewed monthly by engineering managers. Items are scored by **risk** (1–5) × **cost of delay** (1–5).

| ID | Item | Owner | Risk | Delay | Score | Status |
|---|---|---|---|---|---|---|
| TD-001 | Self-managed ClickHouse; manual, risky upgrades | Data Infra | 4 | 4 | 16 | Evaluation planned Q2 2027 |
| TD-002 | Beacon API v1 endpoints without cursor pagination | Beacon Product | 3 | 3 | 9 | Retiring 30 Jun 2027 |
| TD-003 | `ledger-invoicer` single-region deployment | Ledger Product | 4 | 3 | 12 | Design doc in review |
| TD-004 | Terraform storage modules refactored without tests (INC-2041 root cause) | Platform | 5 | 4 | 20 | Tests + policy checks added; module tests ongoing |
| TD-005 | Feature flags past expiry (37 flags) | All teams | 2 | 2 | 4 | Weekly clean-up |
| TD-006 | Alert rules evaluated by a single `beacon-alerts` leader | Beacon Product | 3 | 3 | 9 | Sharding planned |
| TD-007 | Duplicate tenancy logic in `ledger-connectors` | Ledger Product | 4 | 2 | 8 | Move to shared library |
| TD-008 | Node.js 18 still used by two Ledger services (EOL) | Ledger Product | 3 | 4 | 12 | Upgrade to Node 22 in Q1 |
| TD-009 | Flaky integration tests in `beacon-processor` | Beacon Product | 2 | 3 | 6 | Quarantined |
| TD-010 | Posture scanner runs every 8 hours instead of continuously | Security | 4 | 4 | 16 | In progress (INC-2041 AI-8) |

## Rules

- Each team spends about **20% of sprint capacity** on items in this register.
- Any item scoring **16 or above** must have a dated plan.
- New items are added by any engineer through a PR to this page.
