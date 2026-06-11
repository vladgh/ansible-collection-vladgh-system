# Ansible Role: Packages

Vlad's Packages Ansible Role.

## Requirements

*_N/A_*

## Role Variables

Available variables are listed below, along with default values (see defaults/main.yml)

### Extra APT repositories on Debian systems

Check <https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/deb822_repository_module.html> for a complete list of parameters

```yaml
packages_apt_extra_repositories:
  - name: docker
    key_url: https://download.docker.com/linux/debian/gpg
    key_filename: docker.asc
    uris:
      - https://download.docker.com/linux/debian
    suites:
      - "{{ ansible_distribution_release }}"
    components:
      - stable
    architectures:
      - amd64
  - name: PBS Community
    repo: "deb http://download.proxmox.com/debian/pbs {{ ansible_facts['distribution_release'] }} pbs-no-subscription"
    repo_filename: pbs-no-subscription
    state: present
```

### Additional packages

- `packages_extra`: A list of packages to install using APT/YUM

### Unattended-upgrades

```yaml
packages_unattended_upgrades_autoupdate_enabled: true
packages_unattended_upgrades_autoupdate_reboot: false
packages_unattended_upgrades_autoupdate_reboot_time: "03:33"
packages_unattended_upgrades_autoupdate_mail_to: ""
packages_unattended_upgrades_autoupdate_mail_on_error: no
```

## Dependencies

*_N/A_*

## Example Playbook

```yaml
- hosts: all
  become: true
  roles:
    - vladgh.system.packages
```
