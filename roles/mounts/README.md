# Ansible Role: Mounts

Vlad's Storage Mounts Ansible Role.

## Requirements

*_N/A_*

## Role Variables

Available variables are listed below, along with default values (see defaults/main.yml)

Check <https://docs.ansible.com/ansible/latest/modules/mount_module.html> for a complete list of parameters

Using FSTAB entries

```yaml
mounts_entries:
  - name: MyBackup
    path: /data/backup
    src: UUID=1234-1234-1234-1234-1234
    fstype: ext4
    state: mounted
  - name: NFS Share
    src: 192.168.1.1:/share
    path: /mnt/share
    fstype: nfs
    opts: defaults
    state: mounted
```

Using SystemD mount points (recommended)

```yaml
mounts_systemd:
  - name: NFS Share
    what: 192.168.1.1:/share
    where: /mnt/share
    type: nfs
    options: defaults
```

## Dependencies

*_N/A_*

## Example Playbook

```yaml
- hosts: all
  become: true
  roles:
    - vladgh.system.mounts
```
