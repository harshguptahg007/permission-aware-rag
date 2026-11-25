---
doc_id: eng-secrets-management
title: Secrets Management
folder: eng
sensitivity: internal
---

# Secrets Management

## Where secrets live

- All production and staging secrets live in **AWS Secrets Manager**, in the account of the environment that uses them.
- Kubernetes workloads read secrets through the **External Secrets Operator**, which syncs them into Kubernetes secrets mounted as files.
- Local development uses fake values in `.env.example`. Real `.env` files are git-ignored and must never contain production secrets.

## Rules

1. **Never commit secrets.** The pre-commit **gitleaks** hook and a CI scan block known secret patterns.
2. Never paste secrets into Slack, tickets, documents or AI tools.
3. Never log secrets (see Coding Standards).
4. Every secret has an **owner team** tagged in Secrets Manager.

## Rotation

| Secret type | Rotation |
|---|---|
| Database passwords | Automatic, every **90 days** |
| Third-party API keys | Every **90 days**, or immediately after someone with access leaves |
| Signed-URL signing keys (Beacon exports) | Every **30 days**; rotated during INC-2041 |
| Customer API keys | Managed by customers; we support two active keys for zero-downtime rotation |
| TLS certificates | Automatic via cert-manager |

## If a secret leaks

1. Treat it as a **SEV2** at minimum (SEV1 if it gives production data access).
2. **Rotate first**, investigate second.
3. Post in `#security` and page Security on-call.
4. Check access logs for use of the leaked secret.
5. Write a postmortem.

## Access

- Engineers can read secret **names** and metadata. Reading secret **values** in production needs a time-bound elevated role and is logged.
- Contractors never get access to production secrets.
