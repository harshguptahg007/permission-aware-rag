---
doc_id: eng-onboarding
title: Engineering Onboarding Guide
folder: eng
sensitivity: internal
---

# Engineering Onboarding Guide

For new **Acme employees** joining Engineering. (Contractors have a separate guide.)

Read the company-wide *Welcome to Acme* guide first. This page covers engineering-specific setup.

## Week 1: Access and setup

1. **GitHub:** your manager requests access to the `acme` GitHub organisation. You'll get the `engineering` team, which has read access to all service repositories and write access to your team's repositories.
2. **Laptop setup:** clone the `dev-setup` repository and run `./bootstrap.sh`. It installs Go, Node.js (via `fnm`), Docker, `kubectl`, `terraform`, the AWS CLI and pre-commit hooks (including **gitleaks**).
3. **AWS:** you get access to the `acme-dev` account through IAM Identity Center on day 1. `staging` read access comes after your first week. **Production access** is granted after 6 weeks and the production-access training.
4. **Local environment:** `make dev-up` in each service repository starts dependencies (Postgres, Kafka, ClickHouse) with Docker Compose.
5. **Slack:** join `#eng`, `#eng-announce`, `#deploys`, `#incidents`, `#platform`, and your team channel.

## Week 2: First change

- Pick an issue labelled `good-first-issue` in your team's repository.
- Ship it to production by the end of week 2, following the *Deploy Runbook*, with your onboarding buddy as reviewer.
- Read the *Architecture Overview*, *Coding Standards* and *API Style Guide*.

## Weeks 3–6: Learning the system

- Attend two incident reviews (they're open to all engineers on Thursdays).
- Read the last three SEV1/SEV2 postmortems, including **INC-2041**.
- Pair with someone from a different team for a day.

## On-call

- You **shadow** on-call after **6 weeks**: you get pages but the primary responds.
- You join the rotation as primary after **3 months**, once your manager signs off.
- See the *On-call Policy* for allowances and expectations.

## Production access rules

- Elevated production roles are time-bound (max 4 hours) and every session is logged.
- Never copy customer data to your laptop.
- Production database queries need a ticket.

## Who to ask

- Your onboarding buddy
- `#eng-help` for anything technical
- Your manager for priorities
