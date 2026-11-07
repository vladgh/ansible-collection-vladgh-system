# semaphore_backup

Back up Semaphore data over SSH using rsync, stopping its service during the copy.

## Requirements

Ansible 2.18 or newer, root privileges, and a Debian-family Linux host with
systemd. Platform-specific tools and service configuration must be available
where indicated below.

## Variables

Defaults are in [defaults/main.yml](defaults/main.yml); input types and required
fields are in [meta/argument_specs.yml](meta/argument_specs.yml).

| Variable | Default |
| --- | --- |
| `semaphore_backup_enabled` | `true` |
| `semaphore_backup_source` | `/opt/semaphore/` |
| `semaphore_backup_schedule` | `*-*-* 03:00:00` |
| `semaphore_backup_service` | `semaphore` |
| `semaphore_backup_script_path` | `/usr/local/bin/semaphore-backup` |
| `semaphore_backup_rsync_opts` | `-a --delete` |
| `semaphore_backup_ssh_timeout` | `30` |
| `semaphore_backup_ssh_key_path` | `/root/.ssh/id_ed25519_semaphore_backup` |

## Usage

```yaml
---
- name: Configure semaphore backup
  hosts: semaphore_backup
  become: true
  roles:
    - vladgh.system.semaphore_backup
```

Put host-specific settings and secret references in inventory. Never commit
credentials to role defaults or examples.

## Operational details

Required inputs are `semaphore_backup_destination` (`backup@backup.example.org:/srv/semaphore/`)
and `semaphore_backup_ssh_private_key` (a reference to encrypted inventory).
The key is stored with mode `0600`, and its task output is suppressed.
`semaphore_backup_rsync_opts` is a whitespace-separated string; shell quoting
inside it is not interpreted. The default `--delete` mirrors source deletions.
The script stops Semaphore and attempts to restart it even if rsync fails.
Set `semaphore_backup_enabled: false` to stop and disable the timer; this does
not interrupt a running backup or remove keys, units, or data. Persistent timers
can run immediately after a missed schedule. Check mode does not test SSH,
backup recovery, or first-install service activation.
