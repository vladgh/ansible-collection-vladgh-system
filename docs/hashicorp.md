# Ansible Role: Hashicorp

![Build Status](https://github.com/vladgh/ansible-role-hashicorp/workflows/CI/badge.svg)

Vlad's Hashicorp Ansible Role. It installs Packer, Terraform and Vault.

## Requirements

*_N/A_*

## Role Variables

Available variables are listed below, along with default values (see defaults/main.yml):

- `packer_version`: Packer version
- `terraform_version`: Terraform version
- `vault_version`: Vault version

## Dependencies

*_N/A_*

## Example Playbook

```yaml
- hosts: all
  become: yes
  roles:
      - vladgh.system.hashicorp
```

## Contribute

See [CONTRIBUTING.md](CONTRIBUTING.md) file.

## License

Licensed under the Apache License, Version 2.0.
See [LICENSE](LICENSE) file.
