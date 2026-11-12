# semaphore

Install Semaphore UI and its Ansible Python environment. Provision its configuration separately.

## Requirements

Ansible 2.21 or newer, root privileges, and a Debian-family Linux host with
systemd. Platform-specific tools and service configuration must be available
where indicated below.

## Variables

Defaults are in [defaults/main.yml](defaults/main.yml); input types and required
fields are in [meta/argument_specs.yml](meta/argument_specs.yml).

| Variable | Default |
| --- | --- |
| `semaphore_version` | `2.19.18` |
| `semaphore_arch` | `{{ 'arm64' if ansible_facts['architecture'] == 'aarch64' else 'amd64' }}` |
| `semaphore_deb_path` | `/tmp/semaphore_{{ semaphore_version }}_linux_{{ semaphore_arch }}.deb` |
| `semaphore_ansible_venv_path` | `/opt/ansible-venv` |
| `semaphore_ansible_python_packages` | `['ansible']` |
| `semaphore_restart_enabled` | `true` |
| `semaphore_config_path` | `/opt/semaphore/config.json` |

## Usage

```yaml
---
- name: Configure semaphore
  hosts: semaphore
  become: true
  roles:
    - vladgh.system.semaphore
```

Put host-specific settings and secret references in inventory. Never commit
credentials to role defaults or examples.

## Operational details

`semaphore_config_path` points to an existing Semaphore JSON configuration,
which this role does not create. Keep database credentials, encryption keys,
and login setup outside the collection. `semaphore_restart_enabled: false`
allows installation before that configuration is provisioned. By default,
package or unit changes schedule a service restart after five seconds, so
Semaphore can finish launching the Ansible job that changed it. The role also
upgrades system packages and installs Python packages into its own environment.
Check mode cannot install missing venv dependencies or validate runtime setup.
