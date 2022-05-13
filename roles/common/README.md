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

```yaml
local_users:
  - name: username
    comment: My User Name
    uid: 8888
    groups: mygroup
    append: yes
    shell: /bin/bash
    authorized_keys:
      - ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIE0yyqRUbBGOW9PcYyuaUMaRi/EFwL59E3wwMn5dJAKQ MyKey
```

### Groups

Check <https://docs.ansible.com/ansible/latest/modules/group_module.html> for a complete list of parameters

```yaml
local_groups:
  - name: mygroup
    gid: 1234
    system: yes
    sudo: yes
```

### Extra APT repositories on Debian systems

Check <https://docs.ansible.com/ansible/latest/modules/apt_repository_module.html> for a complete list of parameters

```yaml
apt_repositories:
  - name: GIT
    repo: ppa:git-core/ppa
```

### CA Certificates

Set to `yes` to install all certificates from `files/ca`

```yaml
install_ca_certificates: yes
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
    opts: defaults
    state: mounted
```

### Security

Install Fail2ban

```yaml
fail2ban_enabled: yes
```

### Miscellaneous

- `disable_lid_switch`: Set to yes to disable lid switch on laptops (defaults to `no`)

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
```

## Dependencies

*_N/A_*

## Example Playbook

```yaml
- hosts: all
  become: yes
  roles:
    - vladgh.system.common
```
