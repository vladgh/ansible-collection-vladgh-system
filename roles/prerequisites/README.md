# prerequisites

Optionally disable PCIe ASPM to work around Intel NUC power management issues.

## Requirements

Ansible 2.21 or newer, root privileges, and a Debian-family Linux host with
systemd. Platform-specific tools and service configuration must be available
where indicated below.

## Variables

Defaults are in [defaults/main.yml](defaults/main.yml); input types and required
fields are in [meta/argument_specs.yml](meta/argument_specs.yml).

| Variable | Default |
| --- | --- |
| `prerequisites_fix_intel_nuc_pm` | `false` |

## Usage

```yaml
---
- name: Configure prerequisites
  hosts: prerequisites
  become: true
  roles:
    - vladgh.system.prerequisites
```

Put host-specific settings and secret references in inventory. Never commit
credentials to role defaults or examples.

## Operational details

`prerequisites_fix_intel_nuc_pm` defaults to `false`, so no boot changes are
made unless enabled. Enabling it writes a GRUB fragment, runs `update-grub`, and
reboots the host. GRUB and `/etc/default/grub.d` must already exist. Existing
`skip_grub` and `skip_reboot` switches suppress those handlers. Setting the option
back to false does not remove the fragment; remove it and regenerate GRUB to
roll back. Check mode cannot validate the resulting boot behavior.

## Verification limits

The Molecule scenario checks installed configuration in a systemd container.
It does not establish behavior on production hardware. First-install check mode
may reference binaries or units not written in check mode; validate a normal
installation in an isolated host before relying on that preview.
