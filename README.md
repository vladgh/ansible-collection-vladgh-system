# Ansible Collection - vladgh.system

Ansible collection that holds roles, that can be used to configure common system services.

## Included content

- **Modules**:
  - [common](roles/common/README.md)
  - [hashicorp](roles/hashicorp/README.md)
  - [logs](roles/logs/README.md)
  - [msmtp](roles/msmtp/README.md)
  - [qemu_guest_agent](roles/qemu_guest_agent/README.md)
  - [shellcheck](roles/shellcheck/README.md)

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

See [Ansible Using collections](https://docs.ansible.com/ansible/latest/user_guide/collections_using.html) for more details.

## Changelog & Releases

This repository keeps a change log using [GitHub's releases][releases]
functionality.

Releases are based on [Semantic Versioning][semver], and use the format
of `MAJOR.MINOR.PATCH`. The version will be incremented
based on the following:

- `MAJOR`: Incompatible or major changes
- `MINOR`: Backwards-compatible new features and enhancements
- `PATCH`: Backwards-compatible bugfixes and package updates

## Contribute

[![Contributor Covenant](https://img.shields.io/badge/Contributor%20Covenant-v2.0%20adopted-ff69b4.svg)](code_of_conduct.md)

Contributions are always welcome! Please read the [contribution guidelines](.github/CONTRIBUTING.md) and the [code of conduct](.github/CODE_OF_CONDUCT.md).

## License

Licensed under the Apache License, Version 2.0.
See [LICENSE](LICENSE) file.
