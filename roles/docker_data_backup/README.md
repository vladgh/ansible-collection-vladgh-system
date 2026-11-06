# docker_data_backup

Installs a script and a systemd timer that stop all running Docker Compose
projects (`docker compose -p <project> stop`, which honors `depends_on` order),
rsync the container data (default `/opt/appdata/`) to a remote host over SSH,
and start the same projects again. The projects are always restarted, even
when the backup fails. Containers started outside Compose (for example with
`docker run`) are left running, so their data may be copied while in use. A short outage during the backup is the price for a
consistent copy of SQLite and other live databases.

## Requirements

- Docker with the Compose plugin installed on the host (for example with `geerlingguy.docker`).
- Compose projects must be discoverable by `docker compose -p <project>` without the compose file.
- A dedicated, restricted account on the destination, with its public key authorized.
- The role is idempotent and supports `--check`.

## Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `docker_data_backup_destination` | required | rsync destination, `user@host:path` |
| `docker_data_backup_ssh_private_key` | required | private key for the destination (keep in the vault) |
| `docker_data_backup_enabled` | `true` | enable the timer |
| `docker_data_backup_source` | `/opt/appdata/` | directory to back up |
| `docker_data_backup_schedule` | `*-*-* 03:30:00` | systemd `OnCalendar` |
| `docker_data_backup_stop_timeout` | `60` | seconds to wait for each container to stop |
| `docker_data_backup_ssh_timeout` | `30` | ssh connect timeout in seconds |
| `docker_data_backup_rsync_opts` | `[-aH, --delete, --numeric-ids]` | rsync options |
| `docker_data_backup_script_path` | `/usr/local/bin/docker-data-backup` | script location |
| `docker_data_backup_ssh_key_path` | `/root/.ssh/id_ed25519_docker_data_backup` | private key location |

## Example playbook

```yaml
- name: Docker data backup
  hosts: docker_data_backup
  roles:
    - { role: vladgh.system.docker_data_backup, tags: ["docker_data_backup"] }
```

```yaml
docker_data_backup_destination: backup@nas.example.com:/mnt/pool/backup/data/
docker_data_backup_ssh_private_key: "{{ vault_docker_data_backup_ssh_private_key }}"
```

## Notes

- `--delete` mirrors deletions. Keep snapshots on the destination for history.
- A non-root destination user cannot set file ownership, so restore ownership
  with `chown` after restoring.
- Run now: `sudo systemctl start docker-data-backup.service`; logs: `journalctl -u docker-data-backup`.
- Rollback: set `docker_data_backup_enabled: false` and run the role. Remove
  the files under `/usr/local/bin`, `/etc/systemd/system` and `/root/.ssh` by hand if needed.

## Operational details

The private key is written with
mode `0600` and task output is suppressed. Tests use an inert key and disabled
timers; they do not prove real SSH or Docker recovery on your hosts.
