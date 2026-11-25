---
doc_id: fin-revenue-recognition
title: Revenue Recognition Policy
folder: finance
sensitivity: internal
---

# Revenue Recognition Policy

**Owner:** Controller
**Framework:** ASC 606 (US GAAP)

## Five-step model, applied to Acme

1. **Identify the contract:** order form + Master Subscription Agreement (MSA), signed by both parties.
2. **Identify performance obligations:** typically (a) the SaaS subscription, (b) usage above committed amounts, (c) professional services (onboarding), if sold.
3. **Determine the transaction price:** fixed subscription fees plus estimated variable usage, constrained to amounts unlikely to reverse.
4. **Allocate the price** to obligations based on standalone selling prices.
5. **Recognise revenue** when (or as) each obligation is satisfied.

## Acme Beacon

- Subscription fees are recognised **ratably** over the subscription term, starting when the customer's tenant is provisioned.
- Overage (ingest above the committed volume) is recognised **in the month the usage occurs**.
- Multi-year prepaid contracts: revenue still recognised ratably; the prepayment is deferred revenue.

## Acme Ledger

- Platform fees: ratably over the term.
- Usage-based fees (per invoice or per transaction processed): recognised in the month of usage.
- Payment processing pass-through fees are recorded **net** (Acme is the agent, not the principal).

## Professional services

Fixed-fee onboarding packages are recognised as delivered, measured by hours spent versus estimated hours.

## Discounts and credits

- Service-level credits (for example, if uptime falls below the SLA) reduce revenue in the period they are earned.
- Goodwill credits given to customers affected by an incident are treated as a reduction of revenue when committed. *Credits related to INC-2041 are being tracked by the Controller with Legal.*

## Commissions

Sales commissions on new contracts are capitalised and amortised over **4 years** (expected customer life), under ASC 340-40.

## Deal review

Any contract with non-standard terms (custom acceptance clauses, termination for convenience, most-favoured-customer pricing, future product commitments) must be reviewed by the Controller **before signature**.
