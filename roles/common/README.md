# Ansible Role: Common

Vlad's Common Ansible Role.

## Requirements

*_N/A_*

## Role Variables

Available variables are listed below, along with default values (see defaults/main.yml)

### Timezone

Configure timezone setting (<https://docs.ansible.com/ansible/latest/collections/community/general/timezone_module.html>; <https://en.wikipedia.org/wiki/List_of_tz_database_time_zones#List>)

```yaml
timezone: America/Chicago
```

### Users

Check <https://docs.ansible.com/ansible/latest/modules/user_module.html> for a complete list of parameters
Multiple authorized keys can be specified in a single key string value by separating them by newlines.
This option is not loop aware, so if you use with_ , it will be exclusive per iteration of the loop.

```yaml
local_users:
  - name: username
    comment: My User Name
    uid: 8888
    groups: mygroup
    append: true
    shell: /bin/bash
    password: "{{ 'mypassword' | password_hash('sha512', 65534 | random(seed=inventory_hostname) | string ) }}"
    authorized_keys: |
      ssh-ed25519 1234 MyKey
      ssh-ed25519 5678 MyOtherKey
  - name: root
    authorized_keys: |
      ssh-ed25519 1234 MyKey
      ssh-ed25519 5678 MyOtherKey
    authorized_key_path: /etc/pve/priv/authorized_keys
    exclusive: true
```

### Groups

Check <https://docs.ansible.com/ansible/latest/modules/group_module.html> for a complete list of parameters

```yaml
local_groups:
  - name: mygroup
    gid: 1234
    system: true
    sudo: true
```

### Extra APT repositories on Debian systems

Check <https://docs.ansible.com/ansible/latest/modules/apt_repository_module.html> for a complete list of parameters

```yaml
# Add Docker signing key
apt_extra_keys:
  - name: docker key
    url: "https://download.docker.com/linux/{{ ansible_facts['distribution']|lower }}/gpg"
    repo_filename: docker
# Disable PBS Enterprise Repository
apt_disable_repositories:
  - name: PBS Enterprise
    repo: "deb https://enterprise.proxmox.com/debian/pbs {{ ansible_facts['distribution_release'] }} pbs-enterprise"
    repo_filename: pbs-enterprise
    state: absent
# Enable PBS Community Repository
apt_extra_repositories:
  - name: PBS Community
    repo: "deb http://download.proxmox.com/debian/pbs {{ ansible_facts['distribution_release'] }} pbs-no-subscription"
    repo_filename: pbs-no-subscription
    state: present
```

### Additional packages

- `additional_packages`: A list of packages to install using APT/YUM

### Python packages

Check <https://docs.ansible.com/ansible/latest/modules/pip_module.html> for more info

```yaml
# System wide install
pip_install_packages: [ipaddress]
# User install
pip_user_install_packages:
  - user: test
    packages: [colorama]
    state: latest
# Install using PIPX
pipx_packages:
  - yamllint
  - name: ansible
    user: myuser
    install_deps: true
  - name: ansible
    user: myuser
    inject_packages:
      - ansible-lint
    install_apps: true
    state: inject
```

### CA Certificates

Set to `true` to install all certificates from `files/ca`

```yaml
install_ca_certificates: true
```

### BASH Extra Aliases

```yaml
bash_extra_aliases: []
  - alias: ll
    command: ls -lahpF
    dest: /root/.bashrc
    owner: root
    group: root
```

### System

```yaml
sysctl_overwrite:
  # Increase the amount of inotify watchers
  fs.inotify.max_user_watches: 524288
```

### CRON Jobs

Check <https://docs.ansible.com/ansible/latest/modules/cron_module.html> for a complete list of parameters

```yaml
cron_jobs:
  - name: Docker CleanUp
    minute: '2'
    hour: '2'
    job: docker system prune --force 2>&1 | /usr/bin/logger -t DockerCleanUp
```

### Extra mount points

Check <https://docs.ansible.com/ansible/latest/modules/mount_module.html> for a complete list of parameters

```yaml
mounts:
  - name: MyBackup
    path: /data/backup
    src: UUID=1234-1234-1234-1234-1234
    fstype: ext4
    state: mounted
  - name: NFS Share
    src: 192.168.1.1:/share
    path: /mnt/share
    fstype: nfs
    opts: defaults
    state: mounted
```

Using SystemD mount points

```yaml
systemd_mounts:
  - name: NFS Share
    what: 192.168.1.1:/share
    where: /mnt/share
    type: nfs
    options: defaults
```

### Security

Install Fail2ban

```yaml
fail2ban_enabled: true
```

### Extra shell commands

```yaml
shell_extra_commands:
  - name: Test
    cmd: touch test.txt
    chdir: /tmp
    creates: test.txt
```

### Unattended-upgrades

```yaml
unattended_upgrades_autoupdate_enabled: true
unattended_upgrades_autoupdate_reboot: false
unattended_upgrades_autoupdate_reboot_time: "03:33"
unattended_upgrades_autoupdate_mail_to: ""
unattended_upgrades_autoupdate_mail_on_error: no
```

### Miscellaneous

- `disable_lid_switch`: Set to `true` to disable lid switch on laptops (defaults to `false`)

## Dependencies

*_N/A_*

## Example Playbook

```yaml
- hosts: all
  become: true
  roles:
    - vladgh.system.common
```
