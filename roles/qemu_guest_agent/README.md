# Ansible Role: QEMU Guest Agent

Vlad's Ansible Role for QEMU Guest Agent.

## Requirements

*_N/A_*

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`)

```yaml
sanoid_templates:
  default:
    frequently: 4
    hourly: 24
    daily: 7
    monthly: 12
    yearly: 1
    autosnap: 'yes'
    autoprune: 'yes'
sanoid_datasets:
  wpool/code:
    use_template: default
```

## Dependencies

*_N/A_*

## Example Playbook

```yaml
- hosts: all
  become: true
  roles:
      - vladgh.system.qemu_guest_agent
```
