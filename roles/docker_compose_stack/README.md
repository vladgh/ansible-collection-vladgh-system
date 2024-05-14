# Ansible Role: MSMTP

Vlad's Ansible Role for creating Docker Compose stacks.

## Requirements

*_N/A_*

## Role Variables

Available variables are listed below, along with default values (see defaults/main.yml)

- `docker_networks`: A list of networks that need to be created before the stacks
- `docker_compose_stack`: A list of stacks and their environment variables

## Dependencies

*_N/A_*

## Example Playbook

```yaml
- hosts: all
  become: true
  roles:
      - vladgh.system.docker_compose_stack
  vars:
    docker_networks:
      - name: simple_stack
    docker_compose_stack:
      - name: stack_with_variables
          env:
            USER: myname
            PASSWORD: "{{ vault_smtp_password }}"
```
