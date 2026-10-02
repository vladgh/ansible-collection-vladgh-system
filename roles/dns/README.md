# Ansible Role: DNS

Vlad's DNS Ansible Role

## Requirements

*_N/A_*

## Role Variables

### Cloudflare DNS records

```yml
# Use EITHER an API token (preferred) ...
cloudflare_api_token: xxx
# ... OR the account email plus Global API Key
cloudflare_account_email: xxx
cloudflare_account_api_key: xxx

cloudflare_dns_records:
  - zone: example.com
    type: CNAME
    record: www
    value: example.com
    state: absent
```

## Dependencies

*_N/A_*

## Example Playbook

```yaml
- hosts: all
  become: true
  roles:
      - vladgh.system.dns
```
