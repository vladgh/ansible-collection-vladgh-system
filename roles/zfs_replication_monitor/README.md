# zfs_replication_monitor

Check ZFS snapshot freshness and report healthy replication to an Uptime Kuma push endpoint.

## Requirements

Ansible 2.18 or newer, root privileges, and a Debian-family Linux host with
systemd. Platform-specific tools and service configuration must be available
where indicated below.

## Variables

Defaults are in [defaults/main.yml](defaults/main.yml); input types and required
fields are in [meta/argument_specs.yml](meta/argument_specs.yml).

| Variable | Default |
| --- | --- |
| `zfs_replication_monitor_max_age` | `108000` |
| `zfs_replication_monitor_on_calendar` | `*-*-* *:06:00` |
| `zfs_replication_monitors` | `[]` |

## Usage

```yaml
---
- name: Configure zfs replication monitor
  hosts: zfs_replication_monitor
  become: true
  roles:
    - vladgh.system.zfs_replication_monitor
```

Put host-specific settings and secret references in inventory. Never commit
credentials to role defaults or examples.

## Operational details

`zfs_replication_monitors` defaults to `[]`. Each entry needs `name`, `dataset`,
and `kuma_url`; optional fields are `max_age` (seconds) and `on_calendar`.
Supply push URLs through encrypted inventory because they contain credentials.
For example, use `kuma_url: "{{ vault_replication_push_url }}"`.
ZFS and curl must already be installed. This check expects an encrypted dataset
with an available key and at least one snapshot. The URL is stored in a root-only
`0600` environment file, and curl reads it through stdin rather than argv.
Only healthy checks send a heartbeat. Failed checks leave the endpoint to time
out. Timers are persistent and can run after missed schedules. Removing a list
entry leaves its units and private environment file; disable its timer and
remove those files explicitly before removing or renaming an entry. Check mode
does not check datasets, encryption keys, endpoint access, or first-run activation.
