# Ansible Role: QEMU Guest Agent

Vlad's Ansible Role for QEMU Guest Agent.

## Requirements

*_N/A_*

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`)

## Dependencies

*_N/A_*

## Example Playbook

```yaml
- hosts: all
  become: true
  roles:
      - vladgh.system.qemu_guest_agent
```
