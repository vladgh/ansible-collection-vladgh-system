# Ansible Role: Docker Compose Stack

Vlad's Ansible Role for deploying and managing Docker Compose stacks as
SystemD services.

Each stack is deployed as a `docker-compose@<name>.service` instance under
a shared `docker-compose.target`, so you can start/stop all stacks together
or manage them individually.

## Requirements

- Docker and Docker Compose must already be installed on the target host
  (e.g. via `geerlingguy.docker`).
- The Compose file for each stack must exist at
  `{{ playbook_dir }}/files/appstack/<inventory_hostname>/<stack_name>.yml` relative to your
  playbook. This directory is copied verbatim to the host.

## Role Variables

Available variables are listed below, along with default values (see defaults/main.yml)

| Variable                        | Default           | Description                                                                 |
| :------------------------------ | :---------------- | :-------------------------------------------------------------------------- |
| `docker_compose_stack`          | (required)        | List of stack definitions. See below.                                       |
| `docker_compose_stack_networks` | `[]`              | List of Docker networks to create before starting stacks (e.g. `proxy`).   |

### Stack Definition

Each entry in `docker_compose_stack` supports:

| Key                   | Required | Description                                                                                              |
| :-------------------- | :------- | :------------------------------------------------------------------------------------------------------- |
| `name`                | Yes      | Stack name. Must match the .yml file under `files/appstack/<inventory_hostname>`.                                         |
| `env`                 | No       | Dict of environment variables written to `.env` in the stack directory.                                  |
| `require_mount_units` | No       | List of SystemD mount units this stack depends on (e.g. `mnt-media.mount`). Adds `BindsTo=` overrides.  |

> **Note on `.env` files:** Values must not be quoted. Docker Compose reads
> them literally, so `FOO="bar"` sets the value to `"bar"` (with quotes).
> See the [Docker Compose env-file docs](https://docs.docker.com/compose/env-file/#syntax-rules).

## SystemD Integration

The role creates:

- `/etc/systemd/system/docker-compose@.service` — a template service unit
  that runs `docker compose up` for any named stack.
- `/etc/systemd/system/docker-compose.target` — a target that pulls in all
  configured stacks, enabling `systemctl start docker-compose.target` to
  bring up every stack at once.
- `/etc/systemd/system/docker-compose@<name>.service.d/override.conf` —
  per-stack drop-in that adds `Requires=`, `After=`, and `BindsTo=`
  dependencies for any `require_mount_units`.

Control all stacks:

```sh
sudo systemctl start docker-compose.target
sudo systemctl stop docker-compose.target
sudo systemctl restart docker-compose.target
```

Control a single stack:

```sh
sudo systemctl restart docker-compose@media.service
sudo journalctl -u docker-compose@media.service -f
```

## Dependencies

*_N/A_*

## Example Playbook

```yaml
- hosts: docker
  become: true
  roles:
      - vladgh.system.docker_compose_stack
  vars:
    docker_compose_stack_networks:
      - name: simple_stack
    docker_compose_stack:
      - name: simple_stack
      - name: stack_with_variables
          env:
            USER: myname
            PASSWORD: "{{ vault_smtp_password }}"
      - name: stack_with_variables_and_mount
        env:
          TZ: US/Central
          PUID: 8523
          PGID: 8446
        require_mount_units:
          - mnt-media.mount
```

## Failure Handling

If a stack fails to restart after a configuration change, the role records
the failure in `vladgh_system_runtime_failures` (a host fact) rather than
aborting the play. Failed stacks are reported at the end of a run via a
dedicated `docker-compose.target` reporting task in `site.yml`.

To review failures after a run:

```sh
ansible -m debug -a "var=vladgh_system_runtime_failures" all
```
