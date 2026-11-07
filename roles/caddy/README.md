# caddy

Install Caddy with the Cloudflare DNS module and manage reverse proxy sites.

## Requirements

Ansible 2.18 or newer, root privileges, and a Debian-family Linux host with
systemd. Platform-specific tools and service configuration must be available
where indicated below.

## Variables

Defaults are in [defaults/main.yml](defaults/main.yml); input types and required
fields are in [meta/argument_specs.yml](meta/argument_specs.yml).

| Variable | Default |
| --- | --- |
| `caddy_version` | `2.11.9` |
| `caddy_cloudflare_version` | `v0.2.4` |
| `caddy_arch` | `{{ 'arm64' if ansible_facts['architecture'] == 'aarch64' else 'amd64' }}` |
| `caddy_user` | `caddy` |
| `caddy_cloudflare_api_token` | `` |
| `caddy_sites` | `[]` |

## Usage

```yaml
---
- name: Configure caddy
  hosts: caddy
  become: true
  roles:
    - vladgh.system.caddy
```

Put host-specific settings and secret references in inventory. Never commit
credentials to role defaults or examples.

## Operational details

Set `caddy_cloudflare_api_token` from an encrypted inventory variable; the
role requires a nonempty token. Each `caddy_sites` entry requires `name` (for
example `app.example.org`) and `upstream` (for example `127.0.0.1:8080`).
The token is written only to a root-owned `0600` environment file. Caddyfile
validation and credential tasks suppress output. The role downloads a custom
Caddy binary, validates its config, and enables and starts the service. Its
Cloudflare credential grants DNS access: scope it to the required zones.

## Verification limits

The Molecule scenario checks installed configuration in a systemd container.
It does not establish behavior on production hardware. First-install check mode
may reference binaries or units not written in check mode; validate a normal
installation in an isolated host before relying on that preview.
