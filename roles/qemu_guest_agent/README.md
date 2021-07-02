# Ansible Role: QEMU Guest Agent

Vlad's Ansible Role for QEMU Guest Agent.

## Requirements

*_N/A_*

## Role Variables

Available variables are listed below, along with default values (see defaults/main.yml):

- `qemu_guest_agent_enabled`: Set to `yes` to enable the QEMU Guest Agent (defaults to `yes`)

## Dependencies

*_N/A_*

## Example Playbook

```yaml
- hosts: all
  become: yes
  roles:
      - vladgh.system.qemu_guest_agent
```
