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
    autosnap: 'yes'
    autoprune: 'yes'
# The datasetys you want to configure
sanoid_datasets: {}
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
