# Ansible Role: Sanoid

Vlad's Sanoid Ansible Role

## Requirements

*_N/A_*

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`)

```yaml
qemu_guest_agent_enabled: yes  # Set to `yes` to enable the QEMU Guest Agent
```

## Dependencies

*_N/A_*

## Example Playbook

```yaml
- hosts: all
  become: yes
  roles:
      - vladgh.system.sanoid
```
