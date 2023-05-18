# Ansible Role: Shellcheck

Vlad's Shellcheck Ansible Role

<https://github.com/koalaman/shellcheck>

## Requirements

*_N/A_*

## Role Variables

Available variables are listed below, along with default values (see defaults/main.yml):

- `shellcheck_version`: Shellcheck version. One of `latest`, `stable` or a version number (defaults to `stable`)

## Dependencies

*_N/A_*

## Example Playbook

```yaml
- hosts: all
  become: true
  roles:
      - vladgh.system.shellcheck
```
