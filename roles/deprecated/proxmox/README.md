# Ansible Role: PVE

Vlad's Proxmox Virtual Environment Ansible Role

## Requirements

*_N/A_*

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`)

```yaml
proxmox_zfs_tune: false  # Set to `true`to change ZFS ARC min and max below
proxmox_zfs_arc_min: 4294967296
proxmox_zfs_arc_max: 8589934592

proxmox_enable_chrony: false  # Set to `true` to replace SystemD Timesyncd with Chrony

proxmox_cluster_disabled: false  # Set to `true` to stop HA Cluster Services

proxmox_gpu_passthrough_enabled: false  # Set to `true` to turn on GPU Passthrough

proxmox_lvm_thin_pool_autoextend: false  # Set to true to allow change for LVM thinpool auto extend
proxmox_lvm_thin_pool_autoextend_threshold: 80
proxmox_lvm_thin_pool_autoextend_percent: 20
```

## Dependencies

*_N/A_*

## Example Playbook

```yaml
- hosts: all
  become: true
  roles:
      - vladgh.system.pve
```
