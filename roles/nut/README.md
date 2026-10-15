# Ansible Role: NUT

Configures [Network UPS Tools](https://networkupstools.org/) as a server,
standalone host or client.

## Requirements

- Debian or Ubuntu (packages `nut-server`, `nut-client`).
- A mail command (for example from the `msmtp` role) when email notifications are enabled.
- `curl` must already be installed when Telegram notifications are enabled; the role fails if it is missing.

## Role Variables

Defaults are in `defaults/main.yml`, validated by `meta/argument_specs.yml`.

| Variable | Default | Description |
|----------|---------|-------------|
| `nut_mode` | `none` | `none`, `standalone`, `netserver` or `netclient` |
| `nut_host` | `ups@localhost` | UPS to monitor |
| `nut_user` / `nut_password` | `monitor` / `changeme` | upsd credentials (store the password in a vault) |
| `nut_role` | `slave` | upsmon role |
| `nut_powervalue` | `1` | Power supplies fed by the UPS |
| `nut_upsmon_settings` | `{}` | Extra `upsmon.conf` directives, for example `POLLFREQ: 5` |
| `nut_notify_events` | `ONBATT ONLINE LOWBATT FSD COMMBAD COMMOK SHUTDOWN REPLBATT` | Events that run the notification script |
| `nut_upsmon_notifyflags` | `{}` | Per-event `NOTIFYFLAG` override, for example `ONBATT: SYSLOG+WALL` |
| `nut_notify_email_enabled` | `false` | Send email notifications |
| `nut_notify_email_to` | unset | Recipient, required when email is enabled |
| `nut_notify_email_subject_prefix` | `UPS` | Email subject prefix |
| `nut_notify_telegram_enabled` | `false` | Send Telegram notifications |
| `nut_telegram_chat_id` | unset | Required when Telegram is enabled |
| `nut_telegram_bot_token` | unset | Required when Telegram is enabled (use a vault value) |

Server variables (`nut_driver`, `nut_port`, `nut_ups_*`, `nut_upsd_*`) are listed in `defaults/main.yml`.

`upsmon.conf` only contains `MONITOR`, `SHUTDOWNCMD` and `POWERDOWNFLAG`, plus
notification and `nut_upsmon_settings` entries. Everything else uses NUT's
built-in defaults. With both notifiers disabled, no `NOTIFYCMD` is configured
and the notification script and Telegram credentials are removed.

The Telegram token is stored in `/etc/nut/telegram.env` (`0640 root:nut`) and
is passed to `curl` on stdin, so it never appears in the process list.

## Breaking changes

`nut_notification_email` and `nut_upsmon_extra` were removed. Use
`nut_notify_email_enabled` with `nut_notify_email_to`, and
`nut_upsmon_settings` / `nut_upsmon_notifyflags`.

## Dependencies

*_N/A_*

## Example Playbook

```yaml
- hosts: nut_clients
  become: true
  roles:
    - role: vladgh.system.nut
      vars:
        nut_mode: netclient
        nut_host: ups@192.0.2.10
        nut_password: "{{ vault_nut_password }}"
        nut_notify_telegram_enabled: true
        nut_telegram_chat_id: "123456789"
        nut_telegram_bot_token: "{{ vault_nut_telegram_bot_token }}"
```

## Idempotence and rollback

The role is idempotent. To roll back, restore the previous `upsmon.conf`
and restart `nut-client`. Disabling a notifier removes its files.
