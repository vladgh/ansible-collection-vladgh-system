# Ansible Role: PVE

Vlad's Proxmox Virtual Environment Ansible Role

## Requirements

*_N/A_*

## Role Variables

See `defaults/main.yml`

## Dependencies

*_N/A_*

## Example Playbook

```yaml
- hosts: all
  become: yes
  roles:
      - vladgh.system.pve
```
