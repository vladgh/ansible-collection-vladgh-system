# Ansible Role: Logs

Vlad's Logs Ansible Role to set up a remote logging.

## Requirements

*_N/A_*

## Role Variables

Available variables are listed below, along with default values (see defaults/main.yml):

- `remote_logs_server`: Remote logs server (required)
- `remote_logs_port`: Remote logs port (defaults to `10514`)
- `remote_logs_protocol`: Remote logs protocol (`tcp` or `udp`)
- `remote_logs_permitted_peer`: Permitted peer
- `remote_logs_ca_file`: CA file (defaults to `/etc/ssl/certs/ca-certificates.crt`)
- `remote_logs_template`: Remote logs template (containing a `name` and a `string`)

## Dependencies

*_N/A_*

## Example Playbook

[Logz.io](https://logz.io/) example

```yaml
- hosts: all
  become: true
  vars:
    logzio_token: '1234'
    remote_logs_server: 'listener.logz.io'
    remote_logs_port: '5001'
    remote_logs_permitted_peer: '*.logz.io'
    remote_logs_template:
      name: 'logzFormatFileTagName'
      string: '[{{ logzio_token }}] <%pri%>%protocol-version% %timestamp:::date-rfc3339% %HOSTNAME% %app-name% %procid% %msgid% [type=syslog] %msg%\n'
  roles:
    - vladgh.system.logs
```
