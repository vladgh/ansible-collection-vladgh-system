# Ansible Collection - vladgh.system

![Test](https://github.com/vladgh/ansible-collection-vladgh-system/actions/workflows/test.yml/badge.svg)

Ansible collection that holds roles, that can be used to configure common system services.

## Included content

### Roles

See [Roles](roles/)

### Playbooks

See [Playbooks](playbooks/)

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
  become: yes
  roles:
    - role: vladgh.system.common
```

### Playbooks

Before using the playbooks, install the required external roles and collections (modify to your own collections installation path)

```sh
ansible-galaxy install --verbose --force --role-file ~/.ansible/collections/ansible_collections/vladgh/system/collection-requirements.yml`
```

Import playbooks

```yaml
---
- import_playbook: vladgh.system.ansible
```

See [Ansible Using collections](https://docs.ansible.com/ansible/latest/user_guide/collections_using.html) for more details.

## Changelog & Releases

This repository keeps a change log using [GitHub's releases][releases]
functionality.

Releases are based on [Semantic Versioning][semver], and use the format
of `MAJOR.MINOR.PATCH`. The version will be incremented
based on the following:

* `MAJOR`: Incompatible or major changes
* `MINOR`: Backwards-compatible new features and enhancements
* `PATCH`: Backwards-compatible bugfixes and package updates

## Contribute

[![Contributor Covenant](https://img.shields.io/badge/Contributor%20Covenant-v2.0%20adopted-007ba7.svg)](https://www.contributor-covenant.org/version/2/0/code_of_conduct.html)

Contributions are always welcome! Please read the [contribution guidelines](.github/CONTRIBUTING.md) and the [code of conduct](.github/CODE_OF_CONDUCT.md).

## License

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)

Licensed under the Apache License, Version 2.0.
See [LICENSE](LICENSE) file.
