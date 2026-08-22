# fleet-medic 🚑

Keep servers healthy over time. One collection, three treatments, all
reporting to Datadog with a reason for everything they could and could
not fix — built to be **triggered by Datadog through an Ascender job
template**.

```bash
ansible-galaxy collection install mcowser_p.fleet_medic
```

| Role | What it does |
| --- | --- |
| `disk_expand` | Threshold-driven disk growth: tries cleanup first, then free LVM space, then grows the **AWS EBS** or **VMware** disk up a configurable size ladder — `growpart` → `pvresize`/`lvextend` → `xfs_growfs`/`resize2fs`. Single-partition **and** LVM layouts on Linux; single-partition drives on Windows. Hard caps, EBS-cooldown awareness, dry-run. |
| `disk_hygiene` | journald vacuum, old kernels, package caches, aged-path rules, Windows temp/recycle-bin — **dry-run by default**, bytes freed reported, DB/WAL paths never touched (the report *shows* the biggest directories instead). |
| `patch_reboot` | dnf/apt and Windows Update, reboot policy `never/if_needed/always`, and **health gates** after reboot: systemd state, required services, HTTP checks, custom command — defined per host (hostvars **or** an on-host `/etc/fleet-medic/health.yml` the app team owns) with a launch-time override. Rolling orchestration via `serial: 1` (see `playbooks/patch.yml`). |
| `medic_report` | Shared: Datadog Event with outcome + reason taxonomy + full report, optional report file, uniform failure policy. |

## Why it could not fix it — the taxonomy

Failures are never mute: `max_size_reached`, `ebs_cooldown` (EBS allows
one size modification per ~6h — the event says when to retry),
`partition_not_growable` (not the last partition on the disk),
`vg_exhausted_no_driver`, `unsupported_fs`, `cloud_api_denied`,
`no_free_space_after_grow`, `gate_failed:<name>`, `patch_failed`.

## Datadog → Ascender → Datadog loop

See [examples/ascender-setup.md](examples/ascender-setup.md): a disk
monitor webhooks the Ascender launch API with the host as the limit;
the role sweeps every mount; the outcome comes back as a Datadog Event
carrying the whole disk report.

## Access-profiles alignment

Growing in place never remounts, so filesystem ACL grants survive.
When you *attach new volumes* instead (not automated here — by
design), remember the profiles-doc rule: a mount hides the ACLs
beneath it — re-apply the access profile after any mount change.

## Future

`acme-please` — ACME client onboarding per the access-profiles TLS
conventions — is the planned next repo in this family.

## License

Apache-2.0
