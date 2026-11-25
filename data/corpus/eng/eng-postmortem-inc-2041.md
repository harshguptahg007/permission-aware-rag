---
doc_id: eng-postmortem-inc-2041
title: Postmortem – INC-2041 Beacon Export Bucket Exposure
folder: eng
sensitivity: internal
---

# Postmortem: INC-2041, Beacon Export Bucket Exposure

**Severity:** SEV1
**Incident commander:** Bob Mills (Platform)
**Date of incident:** 14 September 2026
**Status:** Closed. Action items tracked below.
**Distribution:** Engineering only. Customer impact details and legal analysis are handled by Legal and are not included here.

This is a blameless postmortem. The goal is to understand how the system allowed this to happen, not to find someone to blame.

## 1. Summary

During a Terraform refactor of the shared storage module, the S3 bucket `beacon-exports-prod` lost its **Block Public Access** setting and received a bucket policy that allowed `s3:GetObject` for any principal. For about **9 hours** (02:10–11:05 UTC on 14 September), any object in the bucket could be downloaded by anyone who knew or guessed its object key.

Export object keys contained the tenant ID and a timestamp, but no random component, which made them guessable for someone who already knew one key.

The issue was detected by our cloud posture scanner at 10:52 UTC and fixed at 11:05 UTC.

## 2. Impact

- Bucket `beacon-exports-prod` (prod-us only). `beacon-exports-prod-eu` was **not** affected because it uses a different module version.
- S3 server access logs show **112 object downloads** during the window from IP addresses not belonging to Acme or to a known customer. These downloads came from **3 IP addresses**.
- The downloaded objects belonged to a subset of tenants. The tenant list and customer notification are owned by Legal (see the legal review).
- No credentials, API keys or payment data are stored in export files. Export files contain customer log data, which may include anything customers log.

## 3. Timeline (UTC)

| Time | Event |
|---|---|
| 13 Sep 21:30 | PR #8812 in `infra` (storage module refactor) approved by one reviewer |
| 14 Sep 02:05 | Scheduled Terraform apply runs for `prod-us` |
| 14 Sep 02:10 | Bucket policy updated; Block Public Access removed |
| 14 Sep 03:41 | First download from an unknown IP address |
| 14 Sep 10:52 | Cloud posture scanner raises a *critical* finding: public bucket |
| 14 Sep 10:55 | Security on-call pages Platform on-call; SEV1 declared |
| 14 Sep 11:05 | Block Public Access re-enabled manually; public policy statement removed |
| 14 Sep 11:40 | Signed-URL signing keys rotated |
| 14 Sep 14:00 | Access log analysis starts; Legal informed |
| 14 Sep 19:30 | Terraform fixed and applied; drift resolved |
| 18 Sep | Customer notice published |

## 4. Root cause

### 4.1 The change
PR #8812 consolidated three storage modules into one. The new module had a variable `public_read` with a default of `true`, copied from the module used for the public marketing assets bucket. The `beacon-exports-prod` call site did not set the variable, so it inherited `true`.

The same module also set `block_public_access = var.public_read ? false : true`, so the default removed Block Public Access.

### 4.2 Why review didn't catch it
- The PR diff showed only module code. The effect on `beacon-exports-prod` was visible only in the `terraform plan` output, which was 2,400 lines long. The reviewer read the code diff, not the plan.
- There was no automated policy check on Terraform plans for storage permissions.
- Block Public Access was enforced at bucket level, not at the **account** level.

### 4.3 Why detection took 8 hours
- The cloud posture scanner ran every **8 hours**, not continuously.
- There was no alert on bucket policy changes in production.
- The download traffic was small and didn't trigger anomaly alerts.

## 5. What went well

- Once detected, containment took 13 minutes.
- S3 server access logs were enabled, so we could tell exactly which objects were downloaded and from where.
- The EU bucket was isolated by design.

## 6. What went badly

- Unsafe default in a shared module.
- One reviewer, no plan review, no policy check.
- 8-hour scanner interval.
- Guessable object keys.

## 7. Action items

| ID | Action | Owner | Status |
|---|---|---|---|
| AI-1 | Enable S3 Block Public Access at **account level** in all prod accounts | Platform | Done (15 Sep) |
| AI-2 | OPA/conftest policy check in Terraform CI blocking public buckets | Platform | Done (22 Sep) |
| AI-3 | Require a Security reviewer for IAM/S3/KMS/security-group changes | Eng leadership | Done (20 Sep) |
| AI-4 | Real-time alert on `PutBucketPolicy` and `DeletePublicAccessBlock` events | Security | Done (25 Sep) |
| AI-5 | Add a random 128-bit component to export object keys | Beacon Product | Done (2 Oct) |
| AI-6 | Shorten signed-URL lifetime from 1 hour to 15 minutes | Beacon Product | Done (2 Oct) |
| AI-7 | Change module defaults so every bucket is private unless explicitly overridden | Platform | Done (24 Sep) |
| AI-8 | Run posture scanner continuously (event-driven) | Security | In progress, target Dec 2026 |
| AI-9 | Summarise `terraform plan` diffs per resource in PR comments | Platform | In progress |

## 8. Lessons

1. **Defaults are policy.** A default in a shared module applies to every call site that forgets it.
2. **Review the effect, not the code.** For infrastructure, the plan is the change.
3. **Layer controls.** Account-level guardrails would have stopped this even with a bad module.
