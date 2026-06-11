# Ansible Role: Common

Vlad's Common Ansible Role.

## Requirements

*_N/A_*

## Role Variables

Available variables are listed below, along with default values (see defaults/main.yml)

### Timezone

Configure timezone setting (<https://docs.ansible.com/ansible/latest/collections/community/general/timezone_module.html>; <https://en.wikipedia.org/wiki/List_of_tz_database_time_zones#List>)

```yaml
common_timezone: America/Chicago
```

### CRON Jobs

Check <https://docs.ansible.com/ansible/latest/modules/cron_module.html> for a complete list of parameters

```yaml
common_cron_jobs:
  - name: Docker CleanUp
    minute: '2'
    hour: '2'
    job: docker system prune --force 2>&1 | /usr/bin/logger -t DockerCleanUp
```

### System

```yaml
common_sysctl_overwrite:
  # Increase the amount of inotify watchers
  fs.inotify.max_user_watches: 524288
```

### BASH Extra Aliases

```yaml
common_bash_extra_aliases: []
  - alias: ll
    command: ls -lahpF
    dest: /root/.bashrc
    owner: root
    group: root
```

### Extra shell commands

```yaml
common_shell_extra_commands:
  - name: Test
    cmd: touch test.txt
    chdir: /tmp
    creates: test.txt
```

## Dependencies

*_N/A_*

## Example Playbook

```yaml
- hosts: all
  become: true
  roles:
    - vladgh.system.common
```
