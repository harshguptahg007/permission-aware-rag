---
doc_id: legal-ip-assignment-policy
title: Intellectual Property Assignment and Open Source Policy
folder: legal
sensitivity: confidential
---

# Intellectual Property Assignment and Open Source Policy

**Owner:** Legal

## 1. Ownership of work

- Everything employees create in the course of their work, or using Acme resources, belongs to Acme. This is set out in the **Proprietary Information and Inventions Assignment Agreement (PIIAA)** every employee signs before starting.
- **Contractors** assign IP through their agency agreement. The Northbridge Consulting master services agreement includes a full IP assignment to Acme for all deliverables, including the Corvid connector and `ledger-partner-sdk`.
- Employees can list prior inventions they own in the PIIAA. Anything not listed is presumed to be created at Acme if related to Acme's business.

## 2. Side projects

- Side projects on your own time and equipment, **unrelated to Acme's business**, belong to you.
- If a side project could relate to observability, logging, billing or invoicing, ask Legal for a written release **before** starting.
- California and other states limit employer claims on unrelated inventions; Legal applies those rules where they apply.

## 3. Open source: using it

| Licence | Allowed in shipped code? |
|---|---|
| MIT, BSD, Apache 2.0, ISC | Yes |
| MPL 2.0, LGPL (dynamic linking) | Yes, with Legal review |
| GPL, AGPL, SSPL | **No** |
| No licence / unknown | No |

An automated licence scan runs in CI. Exceptions need written Legal approval.

## 4. Open source: contributing

- Bug fixes to upstream projects we use: allowed, with manager approval.
- New open-source projects from Acme: need Legal and VP Engineering approval.
- Never contribute code that contains Acme secrets, customer data or unreleased product features.

## 5. Patents

- Acme files patents selectively. Engineers who believe they have a patentable invention should fill in the invention disclosure form; Acme pays an **$1,500 award** on filing and $3,000 on grant.
- **Do not search for or read third-party patents** without Legal guidance. If you receive a patent letter from anyone, forward it to Legal immediately and don't reply.

## 6. Trademarks

"Acme Beacon" and "Acme Ledger" are registered trademarks in the US, Canada and UK. Use them as adjectives ("Acme Beacon platform"), not verbs.
