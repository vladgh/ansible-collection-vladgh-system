# Ansible Role: Hashicorp

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
