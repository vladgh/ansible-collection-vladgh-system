# zfs_pool_monitor

Report actionable ZFS storage problems to Uptime Kuma using one monitor per host.
Each monitor checks an explicit list of expected pools and sends one combined
result. Requires Linux, systemd, Bash, GNU date, awk, sed, root privileges, and
an installed `zpool`. The role installs curl but does not install or modify ZFS.

## Configuration

| Variable | Default | Meaning |
| --- | --- | --- |
| `zfs_pool_monitor_jobs` | `[]` | Monitor definitions. |
| `zfs_pool_monitor_on_calendar` | `*-*-* *:00/5:00` | Check every five minutes. |
| `zfs_pool_monitor_capacity_warning` | `80` | Warning utilization percentage. |
| `zfs_pool_monitor_capacity_critical` | `90` | Immediate critical percentage. |
| `zfs_pool_monitor_capacity_delay` | `3600` | Seconds above warning before alerting. |
| `zfs_pool_monitor_scrub_max_age` | `3888000` | Maximum scrub age: 45 days. |

Every job requires unique `name`, nonempty unique `pools`, and `kuma_url`.
Optional `on_calendar` overrides the schedule; `enabled` defaults to true.

```yaml
zfs_pool_monitor_jobs:
  - name: storage
    pools: [tank, backup]
    kuma_url: '{{ vault_kuma_storage_url }}'
```

Invoke `vladgh.system.zfs_pool_monitor` with root privileges. Create one Kuma Push
monitor per host, with a ten-minute heartbeat interval and one retry before
notification. Store the full push URL in encrypted host inventory. The script
replaces URL query parameters with its own `status` and `msg`. It sends `down`
with pool-specific reasons or `up` with a healthy result. HTTP failures fail the
service and eventually cause a missing heartbeat. Successfully reporting an
unhealthy pool does not fail the systemd service. Configure Kuma to notify on
state changes and recovery, without recurring reminders. Host dependencies to
avoid duplicate unreachable-host alerts must be configured in Kuma separately.

## Check behavior

- Missing or non-ONLINE pools, known data errors, nonzero device counters, and
  failed or unreadable status are reported immediately. Table parsing handles
  spaces in vdev paths. Feature-upgrade notices alone do not trigger alerts.
- Capacity at or above 80% must persist for an hour; a reading below 80% resets
  that period and clears the warning. Capacity at or above 90% is immediately
  critical. Thresholds use `zpool` allocation, not a dataset quota.
- A completed scrub with errors is critical. No known completed scrub, a scrub
  older than 45 days, or an unreadable scrub date is an alert. The last observed
  completion time is remembered across resilvers/running scrubs. On first use
  during an active scrub, its start time supplies a temporary age baseline;
  it does not prove a previous successful scrub. Pool-level errors are still
  checked during scrubs. All conditions are evaluated on each five-minute run.

This does not check SMART, replication freshness, or encryption-key availability.
Keep those separate. The role only reports; it never clears ZFS errors, starts
scrubs, or repairs pools.

## Files and lifecycle

Each job installs only a root-owned `0700` script at
`/usr/local/bin/zfs-pool-monitor-<name>` plus `.service` and `.timer` units at
`/etc/systemd/system/zfs-pool-monitor-<name>.*`. Pool names, thresholds, and URL
are shell-quoted in the script. There are no separate configuration files.
Credentials are suppressed in Ansible output/diffs and sent to curl through
stdin. Do not enable shell tracing on these scripts.

Systemd manages the `0700` runtime-state directory
`/var/lib/zfs-pool-monitor-<name>`. It contains numeric capacity-warning start
times and observed scrub completion times. State persists across reboot;
deleting it resets capacity grace periods and remembered scrub history.
Persistent timers may run immediately after downtime. A service cannot run
concurrently with itself, and monitoring is safe to repeat.

Set `enabled: false` and reapply to stop a monitor. Removing or renaming a job
does not remove old units, scripts, or state; retire them explicitly. When
migrating from the earlier per-pool interface, replace `pool` with `pools` and
disable the old per-pool timers before enabling the host monitor. Restore
inventory and reapply to roll back. Check mode predicts files but skips timer
activation and cannot validate pool/endpoint access.

Molecule checks permissions, unit configuration, enabled/disabled timers, and
Bash syntax. Pool-health logic, capacity persistence, scrub-age checks, alert
routing, and notification delivery require separate runtime verification.
