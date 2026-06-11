# Ansible Role: Access

Vlad's Access Ansible Role.

## Requirements

*_N/A_*

## Role Variables

Available variables are listed below, along with default values (see defaults/main.yml)

### Users

Check <https://docs.ansible.com/ansible/latest/modules/user_module.html> for a complete list of parameters
Multiple authorized keys can be specified in a single key string value by separating them by newlines.
This option is not loop aware, so if you use with_ , it will be exclusive per iteration of the loop.

```yaml
access_users:
  - name: username
    comment: My User Name
    uid: 8888
    groups: mygroup
    append: true
    shell: /bin/bash
    password: "{{ 'mypassword' | password_hash('sha512', 65534 | random(seed=inventory_hostname) | string ) }}"
    authorized_keys: |
      ssh-ed25519 1234 MyKey
      ssh-ed25519 5678 MyOtherKey
  - name: root
    authorized_keys: |
      ssh-ed25519 1234 MyKey
      ssh-ed25519 5678 MyOtherKey
    authorized_key_path: /etc/pve/priv/authorized_keys
    exclusive: true
```

### Groups

Check <https://docs.ansible.com/ansible/latest/modules/group_module.html> for a complete list of parameters

```yaml
access_groups:
  - name: mygroup
    gid: 1234
    system: true
    sudo: true
```

## Dependencies

*_N/A_*

## Example Playbook

```yaml
- hosts: all
  become: true
  roles:
    - vladgh.system.access
```
