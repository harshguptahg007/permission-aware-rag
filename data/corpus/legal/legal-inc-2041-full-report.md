---
doc_id: legal-inc-2041-full-report
title: INC-2041 Legal Review and Customer Impact Report (Privileged)
folder: legal
sensitivity: restricted
---

# INC-2041: Legal Review and Customer Impact Report

**PRIVILEGED & CONFIDENTIAL: ATTORNEY–CLIENT COMMUNICATION / ATTORNEY WORK PRODUCT.**
**Prepared by:** Grace Liu, General Counsel, with outside counsel (Kessler Ward LLP)
**Distribution:** General Counsel, CEO, CFO. Do not forward.
**Date:** 30 October 2026

## 1. Purpose

This report records Acme's legal analysis of the INC-2041 storage exposure: which customers were affected, what notification duties apply, contractual exposure, insurance and next steps. The technical root cause is in the engineering postmortem and is not repeated here except where legally relevant.

## 2. Facts relevant to legal analysis

- **Exposure window:** 14 September 2026, 02:10–11:05 UTC (8 hours 55 minutes).
- **Bucket:** `beacon-exports-prod` (US region). The EU/UK bucket was not affected, so **no UK or EU customer data was involved**.
- **Downloads by unknown parties:** 112 export files from 3 IP addresses. Two of the IP addresses belong to a commercial VPN provider; one to a cloud hosting provider. There is no evidence of who was behind them.
- **Affected customers:** **37 Beacon customers** had at least one export file downloaded by an unknown party.
- **Data involved:** customer application logs. Customers control what they log. Our review of the 112 files (performed inside the production environment by Security under Legal direction) found:
  - email addresses and IP addresses of customers' end users in 29 customers' files;
  - **health-related information** (appointment IDs and clinic names) in the files of **Brightwater Health**;
  - partial bank account numbers (last 4 digits) and internal transaction IDs in the files of **Halcyon Bank**;
  - no passwords, Acme API keys or full payment card numbers.

## 3. Affected customers of note

| Customer | ARR | Data concern | Contractual notice period | Notified |
|---|---|---|---|---|
| **Halcyon Bank** | $1.4M | Partial account numbers, transaction IDs | **72 hours** (custom security addendum) | 18 Sep (about **96 hours** after detection) |
| **Brightwater Health** | $0.6M | Health-related data; Business Associate Agreement in place | "Without unreasonable delay", max 60 days under BAA | 17 Sep |
| Northgate Logistics | $0.3M | End-user emails and IPs | Standard MSA: 72 hours | 17 Sep |
| 34 other customers | $2.9M combined | End-user emails and IPs, or none identified | Standard MSA: 72 hours | 17–18 Sep |

## 4. Notification analysis

### 4.1 Regulators
- **HIPAA:** Acme is a business associate of Brightwater Health. Brightwater, as the covered entity, decides on notification to individuals and HHS. We have provided the required information. Brightwater has indicated that fewer than 500 individuals are involved, so media notification is not required; HHS notification may be made in Brightwater's annual log.
- **US state breach laws:** for most customers, the data does not meet the state-law definition of "personal information" (no names combined with account numbers or government IDs). For Halcyon Bank, outside counsel's view is that partial account numbers without access codes **do not** trigger notification in the relevant states. Halcyon disagrees and has notified its own regulator voluntarily.
- **UK/EU (GDPR):** not applicable; no UK/EU data involved.

### 4.2 Contractual
- Halcyon Bank's security addendum requires notice within **72 hours of discovery**. Acme notified at about 96 hours. This is a **likely breach of contract**.
- All other customers were notified within contractual periods.

## 5. Exposure assessment

| Item | Estimate |
|---|---|
| Halcyon Bank claim (breach of contract, remediation costs, possible non-renewal) | **$0.8M – $1.6M** |
| Credits to other customers (goodwill, already committed) | $0.35M |
| Outside counsel and forensics | $0.25M – $0.4M |
| **Total estimated exposure** | **$1.2M – $2.5M** |

### Insurance
- Cyber insurance policy with Ironclad Mutual: $5M limit, **$250,000 retention**.
- Notice of circumstances given to the insurer on 15 September. Insurer has accepted notice and appointed panel counsel (Kessler Ward LLP).

## 6. Litigation hold

A **litigation hold** was issued on 15 September 2026 covering:
- all Slack channels related to the incident (`#inc-2041`, `#security`, `#platform`);
- the `infra` repository history and PR #8812;
- S3 access logs for the affected bucket;
- email of the incident commander, the PR author and reviewer, and the Security on-call.

Automatic deletion has been suspended for these sources. Do not delete anything related to INC-2041.

## 7. Related employee matter

An employee in Platform engineering has raised a concern that they were excluded from work after questioning parts of the incident review (People case **INV-2026-06**). This is being handled as a potential whistleblower-retaliation matter under POL-GEN-009, and I am overseeing it personally. Nothing in this report should be shared with the employees involved.

## 8. Recommendations

1. Offer Halcyon Bank a negotiated settlement: 6 months of service credits (about $0.7M) plus funded independent security assessment, in exchange for a release. Authority requested up to $1.0M.
2. Update the standard MSA so security addenda cannot set notice periods shorter than 72 hours **from confirmation** (not discovery).
3. Add security incident response timing to the quarterly Audit Committee report.
4. Complete the external security firm's storage review and share a summary with key customers under NDA.

## 9. Disclosure in other contexts

- **Crestline Bank:** disclosed under the reporting covenant; not treated as a material adverse change.
- **Project Falcon due diligence:** Lumen Analytics' counsel may ask about security incidents. Disclose INC-2041 at summary level only, under the NDA.
- **Employees:** the company-wide notice (18 September) remains the only approved internal communication.
