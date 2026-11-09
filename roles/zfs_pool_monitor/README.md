# zfs_pool_monitor

Check imported ZFS pools and send healthy heartbeats to Uptime Kuma. Requires
Linux, systemd, Bash, awk, root privileges, and an installed `zpool` command.
The role installs curl; it does not install ZFS or modify pools.

## Configuration

`zfs_pool_monitor_jobs` defaults to `[]`. Each job requires unique `name`, `pool`,
and `kuma_url`. Optional `on_calendar` defaults to
`zfs_pool_monitor_on_calendar` (`*-*-* *:00/5:00`, every five minutes).
Optional `enabled` defaults to true; false stops and disables that timer.

```yaml
zfs_pool_monitor_jobs:
  - name: storage-data
    pool: tank
    kuma_url: '{{ vault_kuma_pool_url }}'
```

Invoke `vladgh.system.zfs_pool_monitor` in a root-privileged play. Create one Kuma
Push monitor per pool with a ten-minute heartbeat interval, and store each full
push URL (with `status=up`) in encrypted host inventory. Each monitor's pool and
URL are embedded in its root-owned `0700` script using shell quoting. There are
no separate configuration files. URLs are passed to curl through stdin and
excluded from task output, diffs, and command arguments. Do not run scripts
with shell tracing, which would expose embedded credentials.

Every check requires the pool to exist, report `ONLINE`, have no known data
errors, and have zero read/write/checksum counters on ONLINE devices. Parsing
uses the C locale. Pool feature-upgrade notices alone do not fail the check.
Missing or unhealthy pools and command failures suppress the heartbeat and fail
the service; Kuma alerts after its heartbeat deadline. Successful curl delivery
does not validate the monitor configuration inside Kuma. Local errors are in
the service journal. This is pool health monitoring, not replication freshness,
capacity, disk SMART, encryption-key availability, or scrub-age monitoring.

Units are `zfs-pool-monitor-<name>.service` and `.timer`; the script is
`/usr/local/bin/zfs-pool-monitor-<name>`. Timers may run immediately
after downtime. Checks are read-only and safe to repeat. Use `enabled: false`
and reapply before retiring a job. Removing or renaming a job does not remove
its existing units or script; remove those explicitly afterward.
Restore inventory and reapply to roll back configuration. Check mode predicts
files but skips timer activation, and cannot verify pool access or delivery.
Molecule verifies installed script permissions, service configuration, timer
enablement, and Bash syntax. It does not exercise pool-health detection or HTTP
delivery; those require separate runtime verification.
