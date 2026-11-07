# ZFS Replication

Configure one scheduled Syncoid ZFS replication job on each host assigned to
this role. The job connection and schedule are provided by the host inventory.

## Requirements

- Ansible 2.18 or newer
- A Debian-family host with APT
- SSH access from the managed host to the destination

## Variables

| Variable | Required | Default | Description |
| --- | --- | --- | --- |
| `zfs_replication_enabled` | No | `true` | Whether to enable and start the timer |
| `zfs_replication_sanoid_version` | No | `v2.3.0` | Syncoid tag to download |
| `zfs_replication_user` | Yes | — | SSH user on the destination |
| `zfs_replication_host` | Yes | — | Destination SSH host |
| `zfs_replication_source_dataset` | Yes | — | Source ZFS dataset |
| `zfs_replication_destination_dataset` | Yes | — | Destination ZFS dataset |
| `zfs_replication_on_calendar` | Yes | — | systemd `OnCalendar` schedule |

Store the required job values in a host-vars file, for example
`inventory/host_vars/<host>/zfs_replication.yml`.

## Example

```yaml
---
- name: Configure ZFS replication
  hosts: zfs_replication
  become: true
  roles:
    - vladgh.system.zfs_replication
```

## Behavior

The role installs Syncoid and its runtime dependencies, then creates and enables
a systemd service and timer. It is idempotent. Set
`zfs_replication_enabled: false` to stop and disable the timer while retaining
its unit files.

The role defaults to a pinned, Renovate-managed Syncoid version.

## Rollback

Disable the timer with `zfs_replication_enabled: false`. The role does not
remove Syncoid or its systemd unit files.

## Operational details

The collection scenario uses a disabled timer and documentation endpoints.
It does not establish ZFS, SSH, or replication correctness on a real host.
