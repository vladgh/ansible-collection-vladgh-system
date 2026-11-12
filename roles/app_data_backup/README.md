# app_data_backup

Scheduled rsync backups of data whose writers must be stopped. Each job stops
systemd services **or** Docker Compose projects, copies to a local directory or
an SSH destination, and restarts its targets even if stopping or copying fails.

## Requirements and behavior

- Debian/Ubuntu or RedHat-family Linux with systemd, Ansible 2.21+, and root access.
  The role installs Bash, coreutils, util-linux, rsync, and an SSH client for remote
  jobs. Storage, applications, Docker, and destination permissions are provisioned
  separately. Compose jobs require a current Compose v2 plugin supporting
  `start --wait --wait-timeout` and `ps --format`.
- A source and a dedicated, existing, writable destination directory. Absolute
  paths are required; remote destinations use `user@host:/path/` (DNS names, IPv4,
  or SSH aliases; use an alias for IPv6). Trailing slashes do not change behavior:
  source **contents** are copied into the destination.
- Remote jobs need rsync, coreutils, and a POSIX shell on the destination. Provision
  the SSH key separately or supply its content through Ansible Vault. Root's SSH
  configuration must trust the destination host key; host verification is strict.
  No remote sudo or destination creation is performed. The destination account
  needs sufficient privileges to preserve requested ownership, ACLs, and xattrs.

All jobs on a host share a blocking lock, so simultaneous schedules queue instead
of skipping backups. They do not coordinate with other roles, other hosts,
deployments, source writers, or destination snapshot jobs.

Before stopping anything, a job checks its paths, optional mountpoints, remote
connectivity, and all shutdown targets. Services must be loaded and stable;
Compose configuration must be valid. Transitional/paused containers cause failure.
Already stopped services and wholly stopped projects are not explicitly restarted. A running
Compose project is restarted as a **whole project**, including any services that
were previously stopped within it. It uses existing containers, not `compose up`.

List targets in **shutdown order**, dependents first. Recovery runs in reverse
order and attempts every restart even after a failure. Compose handles its own
project dependencies; define health checks and `depends_on` appropriately. Services
use their own systemd shutdown timeouts; `stop_timeout` applies only to Compose.
Compose recovery waits for running/healthy state, bounded by `start_timeout`.

Stopping configured writers does not stop unlisted writers or prevent socket,
timer, or external activation. Disable such activation separately, group shared
writers into one job, and avoid concurrent application changes during backups.

## Configuration

```yaml
- name: Configure backups
  hosts: application_hosts
  become: true
  roles:
    - role: vladgh.system.app_data_backup
```

In each host's inventory, configure its jobs. This example illustrates both types;
independent hosts would normally have their own subset:

```yaml
app_data_backup_ssh_private_key: "{{ vault_backup_ssh_private_key }}"
app_data_backup_jobs:
  - name: semaphore
    source: /srv/semaphore/
    destination: backup@nas.example.com:/srv/backups/server-a/semaphore/
    services:
      - semaphore.service
    schedule: '*-*-* 02:00:00'

  - name: appdata
    source: /opt/appdata/
    destination: /srv/backups/server-a/appdata/
    compose_projects:
      - application
      - database
    compose_root: /opt/appstack
    schedule: '*-*-* 03:00:00'
```

Each project must have standard Compose files and its required `.env` under
`<compose_root>/<project_name>/`. Custom `-f` files and interactive-shell-only
configuration are not supported. Non-Compose containers are not discovered.
Use Compose targets for container data; stopping the Docker daemon can leave
containers running when live restore is enabled.

### Shared variables

| Variable | Default | Purpose |
| --- | --- | --- |
| `app_data_backup_jobs` | `[]` | Host's backup jobs |
| `app_data_backup_schedule` | `*-*-* 03:00:00` | Default systemd calendar |
| `app_data_backup_compose_root` | `/opt/appstack` | Default project parent |
| `app_data_backup_stop_timeout` | `60` | Compose graceful stop seconds |
| `app_data_backup_start_timeout` | `120` | Compose recovery wait seconds |
| `app_data_backup_ssh_timeout` | `30` | SSH connection timeout seconds |
| `app_data_backup_ssh_key_path` | `/root/.ssh/id_ed25519_app_data_backup` | Existing or managed key |
| `app_data_backup_ssh_private_key` | unset | Optional Vault-backed key content |
| `app_data_backup_rsync_opts` | `[-aHAX, --numeric-ids, --delete-delay, --protect-args]` | Default rsync options |

Each job requires `name`, `source`, `destination`, and exactly one nonempty
`services` or `compose_projects` list. Job names use lowercase letters, digits,
underscores, and hyphens. Service names end in `.service`.

Optional job fields are `schedule`, `compose_root`, `rsync_opts` (replace the shared
list), `enabled` (default `true`), and `source_mount` (unset). The source mount
guard names an actual mountpoint, not a subdirectory. Destination checks require
an existing, writable directory; they do not verify the destination filesystem.

## Operations, retention, and removal

Each job installs `app-data-backup-<name>.service`, a matching `.timer`, and
`/usr/local/bin/app-data-backup-<name>`. Timers are persistent, so a missed
schedule can run after boot. Jobs run as root with a restrictive umask.

```sh
sudo systemctl start app-data-backup-semaphore.service
sudo journalctl -u app-data-backup-semaphore.service --no-pager
systemctl list-timers --all 'app-data-backup-*'
sudo stat /var/lib/app-data-backup/semaphore.success
```

The success file's modification time advances only after copying **and** service
recovery succeed. A failure leaves the previous success time intact; alert on
failed units and stale/missing success files using your monitoring system.

Default rsync options mirror deletions and preserve numeric owners, permissions,
hard links, ACLs, and xattrs. A failed transfer may leave the mirror partly updated.
Preview a new configuration with rsync's `--dry-run --itemize-changes` before
using destructive options. Keep destination snapshots for history, take them after
successful jobs, and handle offsite replication separately. Test restores.

Downtime lasts for the entire copy. Normal errors, SIGINT, and SIGTERM trigger
recovery. SIGKILL, power loss, or systemd's five-minute stop deadline can interrupt
recovery; inspect and start affected services manually afterward. Recovery failures
make the backup unit fail even if copying succeeded.

Set a job's `enabled: false` and apply the role to disable its timer (an in-progress
copy is allowed to finish). Removing a job from inventory does **not** uninstall
its schedule: disable it first, then stop its unit and remove its script, units,
and optional success file; run `systemctl daemon-reload`. Backup data and shared
SSH keys are never deleted automatically. Roll back by disabling all jobs.
Other backup automation must not run overlapping jobs.

Check mode predicts file/package changes and validates calendars without running
backups; it skips timer configuration to support first-run checks.

## Verification

The Molecule scenario checks installation and permissions, enabled/disabled
schedules, a real local rsync, service recovery, and repeat runs.
From the collection root, run:

```sh
molecule --base-config molecule/base.yml test -s app_data_backup
```

The scenario does not establish actual Docker readiness, SSH permissions, or
restore safety on your hosts.
