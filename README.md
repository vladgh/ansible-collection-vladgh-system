# Ansible Collection - vladgh.system

Ansible collection that holds roles, that can be used to configure common system services.

## Included content

- **Modules**:
  - [common](docs/common.md)
  - [hashicorp](docs/hashicorp.md)
  - [logs](docs/logs.md)
  - [msmtp](docs/msmtp.md)
  - [qemu_guest_agent](docs/qemu_guest_agent.md)
  - [shellcheck](docs/shellcheck.md)

## Using this collection

### Installing the Collection from Ansible Galaxy

Install it with the Ansible Galaxy CLI:

```sh
ansible-galaxy collection install vgh.system
```

You can also include it in a `requirements.yml` file and install it via `ansible-galaxy collection install -r requirements.yml`, using the format:

```yaml
---
collections:
  - name: vladgh.system
```

See [Ansible Using collections](https://docs.ansible.com/ansible/latest/user_guide/collections_using.html) for more details.

## Contribute

[![Contributor Covenant](https://img.shields.io/badge/Contributor%20Covenant-v2.0%20adopted-ff69b4.svg)](code_of_conduct.md)

Contributions are always welcome! Please read the [contribution guidelines](.github/CONTRIBUTING.md) and the [code of conduct](.github/CODE_OF_CONDUCT.md).

## License

Licensed under the Apache License, Version 2.0.
See [LICENSE](LICENSE) file.
