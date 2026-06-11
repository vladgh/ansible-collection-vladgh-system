# Ansible Role: Misc

Vlad's Miscellaneous Ansible Role.

## Requirements

*_N/A_*

## Role Variables

Available variables are listed below, along with default values (see defaults/main.yml)

- `misc_disable_lid_switch`: Set to `true` to disable lid switch on laptops (defaults to `false`)

## Dependencies

*_N/A_*

## Example Playbook

```yaml
- hosts: all
  become: true
  roles:
    - vladgh.system.misc
```
