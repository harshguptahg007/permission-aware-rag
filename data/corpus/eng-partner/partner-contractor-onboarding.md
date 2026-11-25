---
doc_id: partner-contractor-onboarding
title: Contractor Onboarding Guide (Engineering Partners)
folder: eng-partner
sensitivity: internal
---

# Contractor Onboarding Guide

For engineers from partner firms (currently **Northbridge Consulting**) working with Acme Engineering on the Ledger partner integrations.

Acme employees: use the *Engineering Onboarding Guide* instead.

## What you get access to

| System | Access |
|---|---|
| GitHub | `ledger-partner-sdk` repository (write) and `ledger-connectors` (read, PRs only through forks) |
| AWS | `acme-dev` account only, through a contractor role. **No staging or production access.** |
| Slack | Shared channel `#ext-northbridge` and `#ledger-partners`. No access to internal company channels. |
| Documentation | The Engineering partner folder and the API Style Guide |
| Email | No Acme email account. Use your Northbridge email. |

## Setup

1. Your Northbridge manager requests access through Acme's partner manager (Bob Mills, Platform).
2. Sign the Acme contractor confidentiality agreement and complete the security-awareness module.
3. You'll receive a **contractor SSO account** (`firstname.lastname.ext`) with MFA required.
4. Connect through the **Northbridge VPN** to the Acme partner gateway. Direct internet access to Acme dev resources is blocked.
5. Clone `ledger-partner-sdk` and run `make setup`.

## Rules

- **No customer data.** You work only with synthetic test data in `acme-dev`. If you ever see real customer data, stop and tell your Acme contact immediately.
- You can't join the on-call rotation or respond to incidents.
- Code reviews: every PR needs approval from an Acme engineer on the Ledger team.
- Don't store Acme source code outside the approved repositories.

## Time and billing

- Log hours in Northbridge's timesheet system weekly. Acme approves timesheets through Northbridge, not PeopleHub.
- Acme's HR, benefits, leave and expense policies **do not apply** to contractors. Your Northbridge contract does.

## Who to contact

- Day-to-day technical questions: `#ledger-partners`
- Access problems: Bob Mills (Acme partner manager)
- Contract or timesheet questions: your Northbridge manager
