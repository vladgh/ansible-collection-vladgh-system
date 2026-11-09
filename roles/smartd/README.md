# smartd

Manage disk monitoring through `/etc/smartd.conf` and the `smartmontools`
service on Debian and Ubuntu, including SATA and NVMe devices.
Explicit whole-disk paths replace device discovery. Use stable `/dev/disk/by-id/`
paths, not partition paths. Only listed disks are monitored.

## Inputs

| Variable | Default | Purpose |
| --- | --- | --- |
| `smartd_devices` | `[]` | Required nonempty list of disks. |
| `smartd_enabled` | `true` | Enable/start the daemon; false stops/disables it. |
| `smartd_validate_config` | `true` | Validate with `smartd -q showtests` before installation. |
| `smartd_mail_to` | `root` | Mail recipient passed through the packaged smartd runner. |

Each disk has required `path`, optional `type` (default `auto`), `schedule`
(default empty), and `standby` (default true). The type is a smartd `-d` value;
the schedule is its `-s` regular expression. Schedules use the host timezone.
Standby disks skip polling and may defer scheduled tests until awake.
NVMe devices identified by type or path omit the ATA standby directive.
NVMe entries need no extra settings for monitoring; omit `schedule` for
monitoring without self-tests. Test support depends on the installed smartd
version and hardware; the role adds no separate version checks.

```yaml
smartd_devices:
  - path: /dev/disk/by-id/ata-EXAMPLE_disk1
    type: sat
    schedule: '(S/../../1/20|L/../20/./06)'
```

This checks disk health and logs, schedules short tests Mondays at 20:00, and
extended tests on the 20th at 06:00. Tests run in the background and can take
hours. Avoid scheduling extended tests alongside scrubs or heavy replication.
Mail delivery depends on the host's mail setup; the role does not install an
MTA, configure remote alerts, or send test notifications.

Invoke `vladgh.system.smartd` with root privileges. It installs smartmontools,
backs up replaced configuration, and restarts the daemon only on changes.
The validator registers real disks to check configuration and display schedules;
it is not a hardware-independent parser. It does not start scheduled self-tests
or perform the normal health polling pass. Disconnected or unsupported disks
can prevent validation. Disable validation only for isolated rendering tests.

Check mode predicts configuration changes without restarting services. It skips
service checks on first deployment and does not prove device access,
self-test support, or alert delivery. Restore the previous configuration or
inventory and reapply to roll back. Disabling monitoring does not cancel a test
already running inside a drive. No ZFS scrub/TRIM schedules or disk data change.
