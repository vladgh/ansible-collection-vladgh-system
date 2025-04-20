# Ansible Role: Wireguard

Vlad's Wireguard Ansible Role

## Requirements

*_N/A_*

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`)

```yaml
wireguard_config:
  wg0:
    Description: 'Sample WireGuard Server Config'
    Interface:
      PrivateKey: 'WCZYl1008fS5mXwYVNnWWYyF5F1/UWOQ50/Av1WfZEY='
      Address: '10.0.0.1/24'
      ListenPort: '51820'
      PostUp: "iptables -A FORWARD -i %i -j ACCEPT; iptables -t nat -A POSTROUTING -o {{ ansible_facts['default_ipv4']['interface'] }} -j MASQUERADE"
      PostDown: "iptables -D FORWARD -i %i -j ACCEPT; iptables -t nat -D POSTROUTING -o {{ ansible_facts['default_ipv4']['interface'] }} -j MASQUERADE"
    Peers:
      # Client 1
      - PublicKey: 'ALRxvsVybYyj4CBEIpXWeaUKFuPILoT58oFvVDj3KSQ='
        AllowedIPs: '10.0.0.2/32'
      # Client 2
      - PublicKey: '+j+cGiQXFTEFaa51erkOeoWYo/ix/7kvnB7qDmefIgM='
        AllowedIPs: '10.0.0.3/32'
  wg1:
    Description: 'Sample Client Config'
    Interface:
      PrivateKey: 'ACxT753ig3+ulg5x49Pxr3YE4fZ9HexmfveRtGYoR0U='
      Address: '10.0.0.2/24'
    Peers:
      - Endpoint: 'myserver.dyndns.org:51820'
        PublicKey: 'y2DIZQ3oaCYW+bgE9EPxwl6Rd2xTkRRXE9W/VW1ZDg4='
        AllowedIPs: '0.0.0.0/0'
```

## Dependencies

*_N/A_*

## Example Playbook

```yaml
- hosts: all
  become: true
  roles:
      - vladgh.system.wireguard
```
