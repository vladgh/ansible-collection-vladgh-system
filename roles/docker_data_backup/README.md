# docker_data_backup

Installs a script and a systemd timer that discover running Docker Compose
projects, stop them, rsync their data (default `/opt/appdata/`) to a remote host
over SSH, and attempt to restart the projects even if the backup fails.

Compose files must live under `<compose_root>/<project_name>/`, defaulting to
`/opt/appstack/<project_name>/`. Every stop, start, and configuration check uses
`--project-directory` and `-p`, so the service can run from `/`. All discovered
project configurations are validated before any project is stopped.

Use standard Compose filenames, such as `compose.yaml` or `docker-compose.yml`,
and keep required `.env` settings in the project directory. Custom file names,
extra `-f` files, and environments supplied only by an interactive shell are not
supported. Projects already fully stopped and containers outside Compose are
left alone. Within a selected project, `compose start` can start services that
were stopped before the backup. Avoid concurrent deployments during backups.
A short outage allows copying live database files while services are stopped.

## Requirements

- A running Docker Engine, Docker CLI, and Compose plugin.
- Each running project has valid Compose configuration under its named directory.
- A dedicated, restricted account on the destination, with its public key authorized.
- The role is idempotent and supports `--check`.

## Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `docker_data_backup_destination` | required | rsync destination, `user@host:path` |
| `docker_data_backup_ssh_private_key` | required | private key for the destination (keep in the vault) |
| `docker_data_backup_enabled` | `true` | enable the timer |
| `docker_data_backup_compose_root` | `/opt/appstack` | Parent directory for Compose project directories |
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
