---
doc_id: company-security-incident-notice
title: Security Incident Notice – INC-2041
folder: company
sensitivity: public
---

# Security Incident Notice: INC-2041

*Internal copy of the customer-facing notice published on 18 September 2026.*

## What happened

On **14 September 2026**, a storage bucket used to hold **Acme Beacon log export files** was misconfigured during a routine infrastructure change. For about **9 hours**, some export files in that bucket could be accessed by someone who had the exact file URL.

We found the issue through our own monitoring and fixed it the same day.

## Who was affected

A **limited number of Beacon customers** who had generated log exports during the affected window. We have contacted every affected customer directly.

## What information was involved

Log export files, which may contain application log data that customers send to Beacon. **No Acme account passwords, API keys or payment information were exposed.**

## What we have done

- Closed public access to the bucket and rotated the signed-URL keys.
- Made all new storage buckets **private by default** at the infrastructure level.
- Added an automated check that blocks any change making a bucket public.
- Engaged an external security firm to review our storage configuration.

## What employees should do

- **Do not discuss the incident with customers or on social media.** Point any customer questions to their account manager, who has an approved response.
- Press enquiries go to the communications team only.
- Questions from employees can go to **#security** on Slack.

The engineering postmortem and the legal review are tracked separately and are not shared company-wide.
