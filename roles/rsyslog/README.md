# rsyslog

Install rsyslog and configure remote log forwarding.

## Requirements

Ansible 2.18 or newer, root privileges, and a Debian-family Linux host with
systemd. Platform-specific tools and service configuration must be available
where indicated below.

## Variables

Defaults are in [defaults/main.yml](defaults/main.yml); input types and required
fields are in [meta/argument_specs.yml](meta/argument_specs.yml).

| Variable | Default |
| --- | --- |
| `rsyslog_target` | `` |
| `rsyslog_port` | `1514` |
| `rsyslog_protocol` | `udp` |
| `rsyslog_template` | `RSYSLOG_SyslogProtocol23Format` |
| `rsyslog_ruleset` | `*.*` |

## Usage

```yaml
---
- name: Configure rsyslog
  hosts: rsyslog
  become: true
  roles:
    - vladgh.system.rsyslog
```

Put host-specific settings and secret references in inventory. Never commit
credentials to role defaults or examples.

## Operational details

Set a nonempty `rsyslog_target` to the log receiver (for example
`logs.example.org`). `rsyslog_protocol` accepts `udp` or `tcp`; this role does
not configure TLS. Ensure sensitive logs travel only across a trusted network.
Configuration changes restart rsyslog. The role retains its forwarding file
until explicitly removed; an empty target does not disable forwarding.

## Verification limits

The Molecule scenario checks installed configuration in a systemd container.
It does not establish behavior on production hardware. First-install check mode
may reference binaries or units not written in check mode; validate a normal
installation in an isolated host before relying on that preview.
