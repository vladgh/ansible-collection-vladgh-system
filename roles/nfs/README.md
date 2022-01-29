# Ansible Role: NFS

Vlad's NFS Server Ansible Role.

## Requirements

*_N/A_*

## Role Variables

Available variables are listed below, along with default values (see defaults/main.yml)

- `nfs_exports`: A list of exports which will be placed in the /etc/exports file. See [Ubuntu's simple Network File System (NFS) guide](https://ubuntu.com/server/docs/service-nfs) for more info and examples.

```yaml
nfs_exports:
  # Basic
  - '/home/public  *(rw,async,no_root_squash)'
  # Similar to Synology (only allow 192.168.1.0/24)
  - '/volume1/example  192.168.1.0/24(rw,async,no_wdelay,no_root_squash,insecure_locks,sec=sys,anonuid=1025,anongid=100)'
```

## Dependencies

*_N/A_*

## Example Playbook

```yaml
- hosts: nfs_servers
  become: yes
  roles:
      - vladgh.system.nfs
  vars:
    nfs_exports:
      - '/home/public *(rw,async,no_root_squash)'
```
