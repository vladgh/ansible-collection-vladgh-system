# Ansible Role: Shellcheck

![Build Status](https://github.com/vladgh/ansible-role-shellcheck/workflows/CI/badge.svg)

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
  become: yes
  roles:
      - vladgh.system.shellcheck
```

## Contribute

See [CONTRIBUTING.md](CONTRIBUTING.md) file.

## License

Licensed under the Apache License, Version 2.0.
See [LICENSE](LICENSE) file.
