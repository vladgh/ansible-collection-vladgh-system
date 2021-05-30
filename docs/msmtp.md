# Ansible Role: MSMTP

![Build Status](https://github.com/vladgh/ansible-role-common/workflows/CI/badge.svg)

Vlad's MSMTP Ansible Role.

## Requirements

*_N/A_*

## Role Variables

Available variables are listed below, along with default values (see defaults/main.yml)

- `msmtp_enabled`: Set to `yes` to enable MSMTP relay (defaults to `no`)
- `msmtp_log`: Log type; one of `syslog`, `file` or `no` (defaults to `syslog`)
- `msmtp_logfile`: If above type is file, the path to it (defaults to `/var/log/msmtp.log`)
- `msmtp_accounts`: A list of smtp accounts, for example

    ```yaml
    - account: mysmtp
      host: smtp.example.com
      port: 587
      auth: on
      from: me
      user: myuser
      password: 123456
    ```

- `msmtp_default_account`: Default MSMTP account from the list above
- `msmtp_alias_default`: The default user alias (defaults to `root`)
- `msmtp_alias_root`: The root alias (defaults to `msmtp_alias_default`)
- `msmtp_alias_cron`: The cron alias (defaults to `msmtp_alias_default`)
- `msmtp_ca_certificates_bundle`: The path for the certificates bundle (default to system's CA Certificates Paths; see `vars/OS_FAMILY.yml`)

## Dependencies

*_N/A_*

## Example Playbook

``` yaml
# Sendgrid
msmtp_enabled: yes
msmtp_default_account: Sendgrid
msmtp_alias_default: my_verified_sender@example.com
msmtp_accounts:
  - account: Sendgrid
    host: smtp.sendgrid.net
    port: 587
    auth: 'on'  # Comment this value, otherwise it will become "True"
    from: my_verified_sender@example.com
    user: apikey
    password: "{{ vault_msmtp_sendgrid_password }}"
```

## Contribute

[![Contributor Covenant](https://img.shields.io/badge/Contributor%20Covenant-v2.0%20adopted-ff69b4.svg)](code_of_conduct.md)

Contributions are always welcome! Please read the [contribution guidelines](.github/CONTRIBUTING.md) and the [code of conduct](.github/CODE_OF_CONDUCT.md).

## License

Licensed under the Apache License, Version 2.0.
See [LICENSE](LICENSE) file.
