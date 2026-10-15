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

### Server Configuration

Use `nut_mode: netserver` on the host connected to the UPS. This configures
both the NUT server and a local `upsmon` client. For a USB UPS, the defaults
use the `usbhid-ups` driver and automatic port detection; set `nut_driver` and
`nut_port` to match other hardware.

| Variable | Default | Description |
|----------|---------|-------------|
| `nut_driver` | `usbhid-ups` | UPS driver |
| `nut_port` | `auto` | UPS device port |
| `nut_ups_description` | `UPS` | Description written to `ups.conf` |
| `nut_ups_ignorelb` | `false` | Ignore the driver's low-battery flag |
| `nut_ups_override_battery_charge_low` | `25` | Low battery threshold, as a percentage |
| `nut_ups_override_battery_runtime_low` | `300` | Low runtime threshold, in seconds |
| `nut_ups_extra` | `maxretry = 3` | Additional `ups.conf` directives |
| `nut_upsd_extra` | `LISTEN 0.0.0.0 3493` | Additional `upsd.conf` directives |
| `nut_upsd_users` | See `defaults/main.yml` | Accounts and permissions for `upsd` |

`nut_upsd_users` defines the accounts that server and clients use. Give the
local monitor the `master` role and remote monitors the `slave` role. Use
separate passwords for the admin, monitor, and client accounts, and store
password values in Ansible Vault. Do not use the role's example `changeme`
passwords in a deployed inventory.

The default listener binds to every interface on port 3493. Prefer setting
`nut_upsd_extra` to a `LISTEN` directive for the server's client-facing
address, and restrict port 3493 to trusted clients with a firewall.

The server playbook below shows these settings together. Its password fields
use the `nut_*_password` aliases, which should resolve to secret values kept
in Ansible Vault.

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

## Example Playbooks

### NUT Server

```yaml
- name: Configure the UPS server
  hosts: nut_servers
  become: true
  roles:
    - role: vladgh.system.nut
      vars:
        nut_mode: netserver
        nut_driver: usbhid-ups
        nut_port: auto
        nut_ups_description: UPS on the server
        nut_upsd_extra: |
          LISTEN 0.0.0.0 3493
        nut_upsd_users:
          admin:
            password: "{{ vault_nut_admin_password }}"
            actions: SET FSD
            instcmds: ALL
          monitor:
            password: "{{ vault_nut_monitor_password }}"
            upsmon: master
          client:
            password: "{{ vault_nut_client_password }}"
            upsmon: slave
        nut_host: ups@localhost
        nut_user: monitor
        nut_password: "{{ vault_nut_monitor_password }}"
        nut_role: master
        nut_upsmon_settings:
          POLLFREQ: 5
          DEADTIME: 15
          FINALDELAY: 5
          MINSUPPLIES: 1
        nut_notify_email_enabled: true
        nut_notify_email_to: ops@example.com
```

### NUT Client

```yaml
- name: Monitor the UPS from a client
  hosts: nut_clients
  become: true
  roles:
    - role: vladgh.system.nut
      vars:
        nut_mode: netclient
        nut_host: ups@192.0.2.10
        nut_user: client
        nut_password: "{{ vault_nut_client_password }}"
        nut_role: slave
        nut_notify_telegram_enabled: true
        nut_telegram_chat_id: "123456789"
        nut_telegram_bot_token: "{{ vault_nut_telegram_bot_token }}"
```

## Idempotence and rollback

The role is idempotent. To roll back, restore the previous `upsmon.conf`
and restart `nut-client`. Disabling a notifier removes its files.
