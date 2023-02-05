# Ansible Role: DNS

Vlad's DNS Ansible Role

## Requirements

*_N/A_*

## Role Variables

### Cloudflare Dynamic DNS IP Updater

Installs this script  used to update Dynamic DNS (DDNS) service based on Cloudflare!
<https://github.com/K0p1-Git/cloudflare-ddns-updater>

```yml
cloudflare_ddns_updater_enabled: yes  # Set to `yes` to install ddns updater script
cloudflare_ddns_updater_config:
  auth_email: ""           # The email used to login 'https://dash.cloudflare.com'
  auth_method: "token"     # Set to "global" for Global API Key or "token" for Scoped API Token
  auth_key: ""             # Your API Token or Global API Key
  zone_identifier: ""      # Can be found in the "Overview" tab of your domain
  record_name: ""          # Which record you want to be synced
  ttl: "3600"              # Set the DNS TTL (seconds)
  proxy: false             # Set the proxy to true or false
  slacksitename: ""        # Title of site "Example Site"
  slackchannel: ""         # Slack Channel #example
  slackuri: ""             # URI for Slack WebHook "https://hooks.slack.com/services/xxxxx"
```

### Cloudflare DNS records

```yml
cloudflare_email: xxx  # The email used to login 'https://dash.cloudflare.com'
cloudflare_api_token: xxx  # Your API Token or Global API Key
cloudflare_dns_records:
  - zone: example.com
    type: CNAME
    record: www
    value: example.com
    state: absent
```

### Local DNS resolver

```yaml
dns_stub_listener: no  # Set to `no` to remove local stub listener and use the DNS below
dns_resolved: 127.0.0.1  # Space separated list (Ex: 8.8.8.8 8.8.4.4)
```

## Dependencies

*_N/A_*

## Example Playbook

```yaml
- hosts: all
  become: yes
  roles:
      - vladgh.system.dns
```
