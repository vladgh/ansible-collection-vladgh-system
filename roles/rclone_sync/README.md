# rclone_sync

Schedule `rclone sync` jobs with systemd. Each job supplies a source and a
destination, which can be local directories or configured cloud remotes. The
role creates one service and timer per job.

**Sync makes the destination match the source, including deleting destination
files that are absent from the source.** It is one-way synchronization. Start
with `--dry-run` to review proposed changes before transferring or deleting data.

## Requirements

- Linux with systemd and an already installed rclone binary.
- Authorized remotes in an existing rclone config for jobs using cloud storage.
- Root privileges for Ansible to manage system units. Jobs also run as root so
  they can access the configured paths and credentials; grant those credentials
  only the access the jobs need.
- Available source and destination storage. Local paths work the same way on
  ZFS as on other filesystems; the role does not manage datasets or mounts.

Installation and authentication stay outside this role. For OAuth remotes, the
config file and its parent directory must allow root to save refreshed tokens.
Keep credentials in the rclone config, not in job flags or inventory examples.

## Variables

| Variable | Default | Description |
| --- | --- | --- |
| `rclone_sync_binary` | `/usr/local/bin/rclone` | Absolute path to the installed executable. |
| `rclone_sync_config_path` | `/root/.config/rclone/rclone.conf` | Existing config used unless a job overrides it. |
| `rclone_sync_jobs` | `[]` | Jobs to manage. An empty list creates no units. |
| `rclone_sync_randomized_delay` | `15m` | Maximum random delay added to each scheduled run. |
| `rclone_sync_schedule` | `*-*-* 04:00:00` | Default systemd `OnCalendar` expression, using the host's timezone. |

Each entry in `rclone_sync_jobs` accepts:

| Field | Default | Description |
| --- | --- | --- |
| `name` | Required | Unique unit identifier: letters, digits, underscores, or hyphens; start with a letter or digit. |
| `source` | Required | Source directory or remote path, such as `/srv/documents` or `gdrive:Documents`. |
| `destination` | Required | Destination directory or remote path. Use absolute paths for local directories. |
| `config_path` | Global config path | Per-job rclone config override. |
| `enabled` | `true` | Whether to enable and start this job's timer. |
| `flags` | `[]` | Extra rclone arguments, one argument per list item. |
| `randomized_delay` | Global delay | Per-job maximum randomized delay; use `0` for none. |
| `schedule` | Global schedule | Per-job systemd `OnCalendar` expression. |

Remote names must match sections in the selected rclone config. Flags are passed
directly to rclone; shell substitutions, pipes, and redirections are not evaluated.
For a flag with a separate value, use two entries, such as `--exclude` and `*.tmp`.

## Playbook

The existing Rclone play in `site.yml` runs this role after the external installer.
For a standalone playbook, install and configure rclone first, then use:

File: `rclone-sync.yml` (example)

```yaml
---
- name: Schedule file synchronization
  hosts: rclone
  become: true
  roles:
    - role: vladgh.system.rclone_sync
      tags:
        - vladgh.system.rclone_sync
```

`rclone_sync` is distributed in the `vladgh.system` collection. Keep job settings in
inventory host or group variables rather than in the playbook.

## Examples

These are alternative inventory configurations. Combine jobs into one
`rclone_sync_jobs` list when using several examples on the same host. All examples
include `--dry-run`; remove that flag and reapply after reviewing the journal.

### Google Drive and Box to local directories

File: `inventory/host_vars/example/rclone_sync.yml` (example)

```yaml
---
rclone_sync_jobs:
  - name: gdrive
    source: 'gdrive:'
    destination: /srv/cloud/gdrive
    schedule: '*-*-* 04:00:00'
    flags:
      - --dry-run
      - --log-level=INFO
  - name: box
    source: 'box:Documents'
    destination: /srv/cloud/box-documents
    schedule: '*-*-* 05:00:00'
    flags:
      - --dry-run
      - --log-level=INFO
```

### Local directory to a cloud remote

File: `inventory/host_vars/example/rclone_sync.yml` (example)

```yaml
---
rclone_sync_jobs:
  - name: documents-upload
    source: /srv/documents
    destination: 'box:Home/Documents'
    flags:
      - --dry-run
      - --log-level=INFO
      - --exclude
      - '*.tmp'
rclone_sync_schedule: daily
```

### Local directory to another local directory

File: `inventory/host_vars/example/rclone_sync.yml` (example)

```yaml
---
rclone_sync_jobs:
  - name: local-mirror
    source: /srv/documents
    destination: /srv/mirrors/documents
    randomized_delay: '0'
    schedule: hourly
    flags:
      - --dry-run
      - --log-level=INFO
```

Use separate source and destination directories; do not put either inside the
other. Different jobs should have separate, non-overlapping destinations.

### Cloud remote to another cloud remote

File: `inventory/host_vars/example/rclone_sync.yml` (example)

```yaml
---
rclone_sync_jobs:
  - name: cloud-mirror
    source: 'gdrive:Shared'
    destination: 'box:Mirrors/Shared'
    config_path: /root/.config/rclone/cloud.conf
    enabled: false
    flags:
      - --dry-run
      - --log-level=INFO
```

This last example installs the units with the timer disabled. Both remotes must
be authorized in `cloud.conf`. Set `enabled: true` and reapply to schedule it.

## Operation

Units are named `rclone-sync-<name>.service` and `rclone-sync-<name>.timer`.
Inspect a dry run or trigger a job manually with the following commands.

Run on the managed host:

```sh
sudo systemctl start rclone-sync-gdrive.service
sudo journalctl -u rclone-sync-gdrive.service
systemctl list-timers 'rclone-sync-*'
```

Timers are persistent: they catch up a missed scheduled run after downtime.
Enabling a timer can therefore trigger a run immediately. systemd prevents
concurrent runs of the same service; different jobs can run in parallel.
Files created by a job use a restrictive `0077` umask. Transfer output and errors
are recorded in the system journal; `--log-level=INFO` adds transfer details.

## Idempotence and check mode

The role uses idempotent template and systemd modules. Reapplying unchanged
settings is intended to leave units and timer state unchanged; it does not
manually trigger transfers. Runtime and idempotence tests have not yet been added.

First-deployment Ansible check mode is not yet verified: the timer tasks may
reference units that check mode has not written. Ansible `--check` also does not
preview file synchronization. Use the job's rclone `--dry-run` flag for that.

## Disable, remove, or roll back

Set a job's `enabled: false` and reapply to stop future scheduled runs. An active
sync continues; stop its service explicitly if needed.

Disable a job before removing or renaming it in inventory. Removing a job from
`rclone_sync_jobs` does not remove its units or disable its timer. An empty list
also leaves previously installed units untouched.

To roll back configuration, restore the previous inventory values and reapply.
This does not restore files changed or deleted by a sync. Use independently
managed snapshots or backups when historical recovery is required.

## Operational details

Collection test scenarios verify generated units, timer state, and disabled
jobs. They do not authenticate cloud remotes. Supply authorization separately
through a protected rclone config; never put access tokens in `flags`.
