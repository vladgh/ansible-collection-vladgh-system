# Ansible Role: Sanoid

Vlad's Ansible role to install Sanoid, Syncoid and Findoid

## Requirements

*_N/A_*

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`)

```yaml
# Templates can be used to define common behavior
sanoid_templates:
  default:
    frequently: 4
    hourly: 24
    daily: 7
    monthly: 12
    yearly: 1
    autosnap: true
    autoprune: true
# The datasets you want to configure
sanoid_datasets: {}
```

## Dependencies

*_N/A_*

## Example Playbook

```yaml
- hosts: all
  become: true
  roles:
      - vladgh.system.sanoid
```

## Host configuration and replication

Requires a Debian-family host with APT, ZFS, and systemd. Configure each host
through inventory, for example `inventory/host_vars/storage/sanoid.yml`:

```yaml
sanoid_datasets:
  tank/documents:
    use_template: default
    recursive: true
  backup/archive:
    autosnap: false
    autoprune: false

sanoid_enabled: true
sanoid_on_calendar: '*:0/15'
sanoid_syncoid_jobs:
  - name: documents
    source: tank/documents
    destination: backup@backup.example.org:backup/documents
    options:
      - --recursive
    on_calendar: hourly
  - name: archive_pull
    source: backup@archive.example.org:tank/archive
    destination: backup/archive
    on_calendar: daily
    enabled: false
```

`sanoid_templates` and `sanoid_datasets` accept upstream Sanoid configuration
keys. Boolean values render as lowercase `true`/`false`. Existing defaults and
snapshot scheduling remain unchanged. `sanoid_version` defaults to `v2.3.0`.
Set `sanoid_enabled: false` to stop and disable snapshot scheduling, including
on replication-only hosts. `sanoid_on_calendar` defaults to `*:0/15`.

`sanoid_syncoid_jobs` defaults to an empty list. Each job requires a unique name
containing letters, digits, underscores, or hyphens (no leading hyphen), plus
`source`, `destination`, and a systemd `on_calendar` expression. Endpoints use
Syncoid syntax: a local dataset or `user@host:dataset`. `options` is a list of
individual Syncoid arguments, defaulting to empty. Jobs run as root; provision
SSH keys, trusted host keys, and ZFS permissions separately. This role does not
change SSH verification or grant remote privileges.

### Optional SSH private key

Set `sanoid_syncoid_ssh_private_key` from an encrypted Vault variable to install
a root-owned key with mode `0600`. Its default is empty, which leaves SSH files
untouched. `sanoid_syncoid_ssh_key_path` defaults to `/root/.ssh/syncoid`; its
parent directory is managed with mode `0700`. Use a dedicated SSH directory.
Key contents are excluded from task output and diffs. For example:

```yaml
sanoid_syncoid_ssh_private_key: '{{ vault_syncoid_ssh_private_key }}'
sanoid_syncoid_jobs:
  - name: documents
    source: tank/documents
    destination: backup@backup.example.org:backup/documents
    options:
      - '--sshkey={{ sanoid_syncoid_ssh_key_path }}'
    on_calendar: hourly
```

Pass `--sshkey` in each remote job's `options` to select the installed key.
Provision trusted host
keys, remote authorization, and ZFS permissions separately. Passphrase-protected
keys need an agent available to the service; the role does not configure one.
Clearing the key variable leaves the installed file intact. Revoke remote
authorization and remove the file explicitly when retiring a key. Check mode
predicts installation without writing the key.

Each job creates `syncoid-<name>.service` and `.timer`. `enabled` defaults to
`true`; set it to `false` to stop and disable its timer. `persistent` defaults
to `false`, so missed replication runs are skipped. All jobs share a blocking
host-local lock, so different jobs wait for each other. Syncoid options can
change remote datasets; review them before enabling a job.

To remove a job's units, retain an entry with `name` and `state: absent` for one
successful run before deleting it from inventory. Simply omitting a job leaves
its existing units intact. Stopping a timer does not interrupt an active
replication; removal also leaves an active service to finish. Neither disabling
nor removing a job deletes datasets or snapshots. Check mode predicts file and
service changes but cannot validate ZFS/SSH access or execute replication.
