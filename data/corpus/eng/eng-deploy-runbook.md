---
doc_id: eng-deploy-runbook
title: Deploy Runbook
folder: eng
sensitivity: internal
---

# Deploy Runbook

**Owner:** Platform group (Bob Mills)
**Applies to:** All services deployed to `staging`, `prod-us` and `prod-eu`.

This runbook explains how code goes from a merged pull request to production, how canaries work, how to roll back, and when you must not deploy.

## 1. Overview

Acme uses **GitOps**. The desired state of every environment lives in the `deploy` repository. **Argo CD** watches that repository and applies changes to the clusters. **Argo Rollouts** runs canary releases.

You never run `kubectl apply` against production by hand. If you find yourself needing to, stop and page the Platform on-call.

The normal flow:

1. Merge a PR to `main` in the service repository.
2. CI builds the image, runs tests, scans it, signs it and pushes it to ECR with the tag `<service>:<git-sha>`.
3. CI opens an automated PR in the `deploy` repository bumping the image tag for `staging`.
4. The staging PR auto-merges if checks pass. Argo CD deploys to staging.
5. After staging smoke tests pass, CI opens a **production promotion PR**.
6. A second engineer approves the promotion PR. Merging it starts the production canary.

## 2. Before you deploy to production

Checklist (the promotion PR template includes it):

- [ ] The change is behind a feature flag if it changes customer-visible behaviour.
- [ ] Database migrations are **backwards compatible** with the currently running version (expand → migrate → contract).
- [ ] Dashboards and alerts exist for any new endpoint or job.
- [ ] You are around for the next **60 minutes** to watch the rollout.
- [ ] It is not a deploy freeze window (section 6).

For high-risk changes (authentication, tenancy, billing calculations, storage permissions), also:

- [ ] A change ticket `CHG-xxxx` has been opened and linked in the PR.
- [ ] A reviewer from the owning team **and** a reviewer from Security have approved.

## 3. Canary releases

Argo Rollouts shifts traffic in steps:

| Step | Traffic to new version | Minimum pause |
|---|---|---|
| 1 | 5% | 10 minutes |
| 2 | 25% | 15 minutes |
| 3 | 50% | 15 minutes |
| 4 | 100% | none |

At each step, an automated analysis compares the canary with the stable version:

- Error rate (5xx) must not exceed the stable version by more than **0.5 percentage points**.
- p95 latency must not exceed the stable version by more than **20%**.
- For `beacon-processor`, Kafka consumer lag must stay below **60 seconds**.

If any check fails, Argo Rollouts **aborts automatically** and shifts all traffic back to the stable version. You'll get a Slack message in `#deploys`.

Services that process background jobs (for example `ledger-invoicer`) can't be split by traffic. They use a **blue-green** strategy instead: the new version starts with zero work, a single worker is enabled for 15 minutes, then the rest switch over.

## 4. Watching a rollout

- Follow progress in `#deploys`, or run:
  ```
  kubectl argo rollouts get rollout <service> -n <namespace> --watch
  ```
- Check the service dashboard in Beacon (`acme-internal` tenant).
- Watch error logs for your service filtered to the new image tag.

## 5. Rolling back

If something looks wrong, **roll back first and investigate afterwards.** Rolling back is never wrong.

### 5.1 During a canary
Abort the rollout:
```
kubectl argo rollouts abort <service> -n <namespace>
```
Traffic returns to the stable version within about 30 seconds.

### 5.2 After a full rollout
Revert the image tag in the `deploy` repository:
1. Run `make rollback SERVICE=<service> ENV=prod-us` in the `deploy` repo. This opens a revert PR to the previous tag.
2. The revert PR can be merged with **one approval** at any time, including during a freeze.
3. Argo CD syncs within 2 minutes.

Alternatively, in an emergency:
```
kubectl argo rollouts undo <service> -n <namespace>
```
and then fix the `deploy` repository so Argo CD doesn't re-apply the bad version.

### 5.3 Database migrations
Never roll back a migration by hand. Because migrations are backwards compatible, the old code works with the new schema. If a migration itself is broken, page the database on-call.

## 6. Deploy freezes

Production deploys are **not allowed**:

- On **Fridays after 14:00** local time of the deploying engineer, and over weekends.
- On public holidays in the US, Canada or UK.
- During the company **quiet week** (27–31 December).
- During a declared **SEV1 or SEV2** incident, except for changes that fix the incident.
- During freeze windows announced by the VP Engineering in `#eng-announce` (for example, before major customer events).

Exceptions need approval from the Engineering Manager on call **and** a director. Rollbacks are always allowed.

## 7. Infrastructure changes (Terraform)

- Terraform changes follow a separate pipeline in the `infra` repository.
- `terraform plan` output is posted on the PR automatically. Reviewers must read the plan, not just the code diff.
- Any change to S3 bucket policies, IAM policies, security groups or KMS keys requires a **Security reviewer**. This rule was added after INC-2041.
- A policy check (OPA/conftest) blocks any plan that makes a storage bucket public or removes Block Public Access.

## 8. Feature flags

- Flags are managed in the flags service. Every flag has an owner and an expiry date.
- Turning a flag on for 100% of tenants counts as a production change and follows the same freeze rules.
- Remove flags within 30 days of full rollout. Expired flags appear in the weekly tech-debt report.

## 9. Troubleshooting

| Symptom | Likely cause | What to do |
|---|---|---|
| Argo CD shows `OutOfSync` for a long time | Manual change in the cluster | Find and revert the manual change; ask in `#platform` |
| Canary aborted with no errors in logs | Latency regression | Check the p95 panel and recent dependency changes |
| Image pull errors | Image not signed or not in ECR | Re-run the CI build job |
| Migration job stuck | Lock on a large table | Page database on-call, don't kill the job |

## 10. Contacts

- `#deploys`: rollout notifications
- `#platform`: questions about the pipeline
- Platform on-call: PagerDuty service "Platform"
