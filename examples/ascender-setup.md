# Ascender / Datadog wiring

## Job templates
1. Project: this collection (or a playbook repo requiring it).
2. Job template **fleet-medic-expand** → `playbooks/expand.yml`,
   inventory with your fleet, *Prompt on launch* for **Limit**.
   Credentials: machine + (AWS credential for EBS hosts, vCenter
   credential for VMware hosts) + a custom credential injecting
   `datadog_api_key` as an extra var.
3. Execution environment must include `boto3` (AWS) / `pyvmomi`
   (VMware) — both are in the default AAP EE images.

## Datadog → Ascender
1. Datadog monitor on `system.disk.in_use` (e.g. `> 0.8`).
2. Webhook integration pointing at
   `https://<ascender>/api/v2/job_templates/<id>/launch/` with an OAuth2
   token header and payload:
   ```json
   {"limit": "$HOSTNAME", "extra_vars": {"dry_run": false}}
   ```
3. Attach the webhook (`@webhook-ascender-expand`) to the monitor
   message.

## The alert back
The role posts a **Datadog Event** (`fleet_medic:disk_expand`,
`outcome:ok|fail`, `reasons:*` tags) containing the full per-mount
report — before/after sizes and, on failure, a taxonomy code
(`max_size_reached`, `ebs_cooldown`, `partition_not_growable`, …).
Create an Event monitor on `fleet_medic:* outcome:fail` to page a
human with the reason inline. Ascender's own notification templates
(job failed) are the belt-and-braces channel for runs that die before
reporting.
