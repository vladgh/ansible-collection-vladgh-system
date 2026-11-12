# wake_on_lan

Install ethtool and configure systemd Wake-on-LAN units for selected interfaces.

## Requirements

Ansible 2.21 or newer, root privileges, and a Debian-family Linux host with
systemd. Platform-specific tools and service configuration must be available
where indicated below.

## Variables

Defaults are in [defaults/main.yml](defaults/main.yml); input types and required
fields are in [meta/argument_specs.yml](meta/argument_specs.yml).

| Variable | Default |
| --- | --- |
| `wake_on_lan_interfaces` | `[]` |

## Usage

```yaml
---
- name: Configure wake on lan
  hosts: wake_on_lan
  become: true
  roles:
    - vladgh.system.wake_on_lan
```

Put host-specific settings and secret references in inventory. Never commit
credentials to role defaults or examples.

## Operational details

`wake_on_lan_interfaces` is a list of interface names, defaulting to empty.
Hardware and drivers must support Wake-on-LAN. The role enables and starts
`wol@<interface>.service`; an empty list leaves previous units enabled. Disable
old interface units explicitly before removing them from inventory. Tests
verify unit installation; container tests cannot establish hardware behavior.

## Verification limits

The Molecule scenario checks installed configuration in a systemd container.
It does not establish behavior on production hardware. First-install check mode
may reference binaries or units not written in check mode; validate a normal
installation in an isolated host before relying on that preview.
