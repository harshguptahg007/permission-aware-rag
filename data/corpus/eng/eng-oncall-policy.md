---
doc_id: eng-oncall-policy
title: On-call Policy
folder: eng
sensitivity: internal
---

# On-call Policy

**Owner:** VP Engineering

## Rotations

- Each team with production services runs its own rotation in **PagerDuty**.
- Rotations are **one week**, handing off on **Monday at 10:00** local time.
- Each rotation has a **primary** and a **secondary**. The secondary is paged if the primary doesn't acknowledge.
- A team needs at least **5 people** in a rotation. Smaller teams merge rotations.

## Severity levels

| Severity | Definition | Acknowledge within | Example |
|---|---|---|---|
| SEV1 | Major customer impact or security/data exposure | 15 minutes | Ingest down for all tenants; data exposed |
| SEV2 | Significant impact for many customers | 30 minutes | Search latency 5× normal |
| SEV3 | Limited impact or degraded non-critical feature | Next business hours | One alert channel failing |
| SEV4 | No customer impact | Ticket | Flaky internal job |

For SEV1 and SEV2, the on-call engineer becomes **incident commander** until they hand off. Open a channel `#inc-<number>` and post updates every 30 minutes.

## Expectations

- Carry your laptop and have reliable internet while on call.
- Be able to start working within **15 minutes** of a page.
- If you can't cover part of your shift, arrange a swap and update PagerDuty.
- Don't deploy non-urgent changes on your own on-call evenings.

## Compensation

- On-call allowance: **$400 per week** as primary (US), CAD 500 (Canada), £300 (UK). Secondary gets half.
- If you're paged between 22:00 and 07:00, you can start late or take time off the next day.
- Working a public holiday on call earns **one day off in lieu** per holiday.

## After an incident

- SEV1 and SEV2 incidents need a **blameless postmortem** within **5 working days**.
- Postmortems are reviewed in the Thursday incident review.
- Action items get owners and dates and are tracked in the incident tracker.

## Health

On-call should be sustainable. If your team gets more than **5 out-of-hours pages a week** for two weeks in a row, the team must spend the next sprint reducing alert noise.
