---
doc_id: eng-architecture-overview
title: Engineering Architecture Overview
folder: eng
sensitivity: internal
---

# Engineering Architecture Overview

**Owner:** Priya Nair (VP Engineering), maintained by the Platform group
**Audience:** Acme engineers. Not for customers or contractors.
**Last updated:** November 2026

This document describes how Acme Beacon and Acme Ledger are built and deployed. It is the starting point for new engineers and for design reviews. If something here is out of date, open a PR against this page.

## 1. Principles

1. **Boring infrastructure.** We use managed AWS services wherever they are good enough, and we run as little stateful infrastructure ourselves as possible.
2. **One way to deploy.** Every service is deployed through the same GitOps pipeline (see the Deploy Runbook).
3. **Tenant isolation by default.** Every request, row, log line and file carries a `tenant_id`. Code that reads data without a tenant filter fails review.
4. **Private by default.** Since INC-2041, every storage resource is private unless an explicit, reviewed exception exists.
5. **Small services, shared libraries.** Services are split by domain, but common concerns (auth, logging, tenancy, config) live in shared libraries so they are implemented once.

## 2. Environments and regions

| Environment | Purpose | AWS account |
|---|---|---|
| `dev` | Personal and feature environments, ephemeral | `acme-dev` |
| `staging` | Pre-production, mirrors prod topology at smaller scale | `acme-staging` |
| `prod-us` | Production for US and default customers, region `us-east-1` | `acme-prod-us` |
| `prod-eu` | Production for UK/EU customers who need data residency, region `eu-west-2` (London) | `acme-prod-eu` |

Each environment runs on **Amazon EKS** (Kubernetes). Infrastructure is defined in **Terraform**, in the `infra` repository, with one workspace per environment. Nobody applies Terraform from a laptop: all applies run through the CI pipeline after review.

## 3. Acme Beacon

Beacon is an observability platform. Customers install the **Beacon agent** on their hosts or as a Kubernetes DaemonSet. The agent ships logs, metrics and traces over HTTPS to our ingestion endpoint.

### 3.1 Services

| Service | Language | Responsibility |
|---|---|---|
| `beacon-ingest` | Go | Receives agent payloads, authenticates the tenant API key, validates and writes to Kafka |
| `beacon-processor` | Go | Consumes from Kafka, parses and enriches events, writes to ClickHouse |
| `beacon-query` | Go | Serves search and dashboard queries against ClickHouse |
| `beacon-alerts` | Go | Evaluates alert rules every 30 seconds and sends notifications |
| `beacon-export` | Go | Builds log export files on request and writes them to S3 |
| `beacon-web` | TypeScript (React) | Customer-facing web UI |
| `beacon-api` | Go | Public REST API (v1 and v2) used by the UI and customers |

### 3.2 Data flow

1. Agent → `beacon-ingest` (behind an AWS Application Load Balancer).
2. `beacon-ingest` → **Amazon MSK (Kafka)**, topic per data type (`logs`, `metrics`, `traces`), partitioned by `tenant_id`.
3. `beacon-processor` → **ClickHouse** cluster (self-managed on EC2, 6 shards × 2 replicas in `prod-us`).
4. `beacon-query` reads from ClickHouse. Every query is rewritten by the tenancy library to add a `tenant_id = ?` filter.
5. Alert rules and dashboards are stored in **PostgreSQL** (Amazon RDS), the `beacon_meta` database.

### 3.3 Storage and retention

- Hot log data: ClickHouse, retained according to the customer's plan (7, 30, 90 or 365 days). See *Data Retention Implementation*.
- Exports: S3 bucket `beacon-exports-prod` (and `beacon-exports-prod-eu`). Files are downloaded through short-lived signed URLs (15 minutes). The bucket has **S3 Block Public Access** enforced at account level since September 2026.
- Archived data beyond the hot retention: S3 with lifecycle rules to Glacier for Enterprise customers who buy extended retention.

### 3.4 Scale (prod-us, October 2026)

- Peak ingest: about 1.8 million events per second.
- Average daily ingest: about 95 TB uncompressed, about 9 TB after ClickHouse compression.
- p95 search latency: 1.4 seconds for 24-hour queries.

## 4. Acme Ledger

Ledger is a billing and invoicing API. Customers send usage events; Ledger turns them into invoices.

### 4.1 Services

| Service | Language | Responsibility |
|---|---|---|
| `ledger-api` | TypeScript (Node.js, Fastify) | Public REST API, authentication, idempotency |
| `ledger-metering` | TypeScript | Aggregates usage events into billable quantities per billing period |
| `ledger-invoicer` | TypeScript | Generates invoices, applies pricing plans, taxes and credits |
| `ledger-payments` | TypeScript | Payment collection via the payment processor, retries, dunning |
| `ledger-webhooks` | TypeScript | Delivers signed webhook events to customer endpoints |
| `ledger-connectors` | TypeScript | Sync with partner ERP and accounting systems (built with Northbridge Consulting) |

### 4.2 Data

- **PostgreSQL** (Amazon Aurora) is the system of record: customers, plans, subscriptions, invoices.
- Usage events land in **Amazon SQS** queues and are written to a partitioned `usage_events` table.
- Every write that changes money requires an **idempotency key**. The API rejects a repeated key with different parameters.
- Money is stored as integer minor units (cents) with an ISO currency code. Never use floats for money.

### 4.3 Multi-currency (planned)

Today Ledger invoices in USD only. Multi-currency support (target Q1 2027) adds currency per plan and FX rates fetched daily and stored per invoice.

## 5. Shared platform

### 5.1 Identity and access
- Customer users log in through `auth-service` (OIDC). Enterprise tenants can use SAML SSO.
- Service-to-service calls use mTLS inside the mesh (Linkerd), plus short-lived service tokens.
- Engineers access production through **AWS IAM Identity Center** with time-bound elevated roles. Production database access requires a ticket and is logged.

### 5.2 Observability (dogfooding)
We monitor Acme's own systems with Beacon, in a separate internal tenant called `acme-internal`. Paging goes through PagerDuty.

### 5.3 CI/CD
- Source code lives on GitHub. CI runs on GitHub Actions.
- Container images are built in CI, scanned, signed and pushed to Amazon ECR.
- Deployments use **Argo CD** with **Argo Rollouts** for canary releases. See the *Deploy Runbook* for the exact steps.

### 5.4 Secrets
All secrets live in **AWS Secrets Manager** and are mounted at runtime. See *Secrets Management*.

## 6. Security boundaries

- Customer data never leaves production accounts. Debugging with customer data happens inside production through approved tooling, never on laptops.
- Contractors (for example, Northbridge Consulting engineers working on `ledger-connectors`) have access to the `ledger-partner-sdk` repository and the `dev` environment only. They do not have production access.
- Security reviews are required for any new data store, any new public endpoint, and any change to tenancy or authentication code.

## 7. Known weak spots

These are tracked in the *Tech Debt Register*:

- ClickHouse is self-managed and upgrades are manual and risky.
- `beacon-api` v1 still has endpoints without cursor pagination. It is being retired; see the *Legacy API Migration Guide*.
- Terraform modules for storage were refactored quickly in 2025; the INC-2041 root cause came from one of these modules.
- `ledger-invoicer` has a single-region deployment; a regional AWS outage would delay invoice generation.

## 8. Where to go next

- Deploy Runbook: how to ship.
- On-call Policy: how we respond.
- Coding Standards and API Style Guide: how we write code and APIs.
- INC-2041 Postmortem: what went wrong with exports, and what changed.
