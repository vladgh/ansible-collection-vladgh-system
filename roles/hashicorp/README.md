# Ansible Role: Hashicorp

Vlad's Hashicorp Ansible Role. It installs Packer, Terraform and Vault.

## Requirements

*_N/A_*

## Role Variables

*_N/A_*

## Dependencies

*_N/A_*

## Example Playbook

```yaml
- hosts: all
  become: yes
  roles:
      - vladgh.system.hashicorp
```
