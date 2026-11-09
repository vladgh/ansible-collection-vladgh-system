# Ansible Collection - vladgh.system

![Test](https://github.com/vladgh/ansible-collection-vladgh-system/actions/workflows/test.yml/badge.svg)

Ansible collection that holds roles, that can be used to configure common system services.

## Using this collection

### Installing the Collection from Ansible Galaxy

Install it with the Ansible Galaxy CLI:

```sh
ansible-galaxy collection install vladgh.system  --upgrade
```

You can also include it in a `requirements.yml` file and install it via `ansible-galaxy collection install -r requirements.yml`, using the format:

```yaml
---
collections:
  - name: vladgh.system
```

Using the GitHub repository and branch

```yaml
collections:
  - name: https://github.com/vladgh/ansible-collection-vladgh-system
    version: main
    type: git
```

### Roles

```yaml
---
- name: Common
  hosts: all
  become: true
  tasks:
    - name: Include common role
      ansible.builtin.include_role:
        name: vladgh.system.common
```

### Additional roles

The following roles are also available; configure host settings and credentials
in inventory. Each role documents its requirements and operational effects.

- [caddy](roles/caddy/README.md)
- [docker_data_backup](roles/docker_data_backup/README.md)
- [prerequisites](roles/prerequisites/README.md)
- [rclone_sync](roles/rclone_sync/README.md)
- [rsyslog](roles/rsyslog/README.md)
- [smartd](roles/smartd/README.md)
- [semaphore](roles/semaphore/README.md)
- [semaphore_backup](roles/semaphore_backup/README.md)
- [wake_on_lan](roles/wake_on_lan/README.md)
- [sanoid](roles/sanoid/README.md) — snapshot policies and scheduled Syncoid replication
- [zfs_replication_monitor](roles/zfs_replication_monitor/README.md)
- [zfs_pool_monitor](roles/zfs_pool_monitor/README.md)

### Playbooks

Import playbooks

```yaml
---
- name: Upgrade Packages
  ansible.builtin.import_playbook: vladgh.system.upgrade
  tags: ["upgrade"]
```

See [Ansible using collections in a playbook](https://docs.ansible.com/ansible/latest/collections_guide/collections_using_playbooks.html) for more details.

## Testing imported roles

Each additional role has a scenario under `molecule/<role>/` registered in the
existing test workflow. From a collection checkout with Molecule, its Docker
plugin, and a Docker daemon, run:

```sh
molecule --base-config molecule/base.yml test --scenario-name rclone_sync
```

The scenarios use documentation endpoints, disabled backup and replication
timers, and future schedules. They check installation, file modes, generated
configuration, and timer enablement. They do not authenticate real cloud remotes,
exercise physical Wake-on-LAN, reboot hardware, or copy production data.

## Changelog & Releases

This repository keeps a change log using [GitHub's releases](releases)
functionality.

Releases are based on [Semantic Versioning](https://semver.org/), and use the format
of `MAJOR.MINOR.PATCH`. The version will be incremented
based on the following:

* `MAJOR`: Incompatible or major changes
* `MINOR`: Backwards-compatible new features and enhancements
* `PATCH`: Backwards-compatible bugfixes and package updates

## Contribute

[![Contributor Covenant](https://img.shields.io/badge/Contributor%20Covenant-2.1-4baaaa.svg)](.github/CODE_OF_CONDUCT.md)

Contributions are always welcome! Please read the [contribution guidelines](.github/CONTRIBUTING.md) and the [code of conduct](.github/CODE_OF_CONDUCT.md).

## License

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)

Licensed under the Apache License, Version 2.0.
See [LICENSE](LICENSE) file.
