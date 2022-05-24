# Ansible Role: NFS

Vlad's NFS Server Ansible Role.

## Requirements

*_N/A_*

## Role Variables

A list of exports which will be placed in the /etc/exports file. See [Ubuntu's simple Network File System (NFS) guide](https://ubuntu.com/server/docs/service-nfs) for more info and examples.

```yaml
nfs_exports:
  # Basic
  - '/home/public  *(rw,async,no_root_squash,no_subtree_check)'
  # Similar to Synology (only allow 192.168.1.0/24)
  - '/volume1/example  192.168.1.0/24(rw,async,no_wdelay,no_root_squash,no_subtree_check,insecure_locks,sec=sys,anonuid=1025,anongid=100)'
```

## Dependencies

*_N/A_*

## Example Playbook

```yaml
- hosts: nfs_servers
  become: yes
  roles:
      - vladgh.system.nfs
```
