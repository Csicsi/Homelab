# GL.iNet SF1200 Setup Guide

## Overview

The **GL.iNet GL-SF1200 (Slate)** is a dual-band travel router that ships with GL.iNet's customized OpenWrt-based firmware. It provides a polished web UI with full OpenWrt power underneath (UCI, opkg, SSH), making it ideal for homelab automation.

## Initial Setup

1. **Power on the router** and connect via WiFi or Ethernet
2. **Access web UI**: `http://192.168.8.1` (default) or `http://gl-sf1200.lan`
3. **Set admin password** on first boot
4. **Configure WAN**: Connect Ethernet cable from LM1200 to WAN port, verify internet connectivity
5. **Enable SSH**: System → Advanced → Enable SSH access (default port 22)
6. **Set root password**: SSH to router and run `passwd` to set root password for Ansible access

## Key Features

- **OpenWrt-based**: Full UCI config, opkg package manager, SSH access
- **Polished web UI**: Simplified interface for common tasks (DHCP, firewall, VPN, plugins)
- **Ansible-ready**: SSH access allows full automation via Ansible/UCI commands
- **Official support**: GL.iNet maintains firmware updates
- **VLAN capable**: Supports network segmentation when needed

---

## Ansible Automation

### What Ansible Can Do

✅ **Network configuration**: LAN IP, DHCP ranges, DNS settings  
✅ **DHCP reservations**: Static IP assignments for all devices  
✅ **Firewall rules**: Custom zones and port forwarding  
✅ **Package installation**: Install additional tools via opkg  
✅ **Idempotent**: Playbooks can be re-run safely

❌ **Cannot set initial password**: Root password must be set manually first (security requirement)

### Workflow

1. **Manual**: Complete initial setup (5 minutes, no LAN IP change needed)
2. **Deploy SSH key**: `ssh-copy-id root@192.168.8.1`
3. **Ansible**: Configure everything else (network, DHCP, firewall, packages, etc.)

See `ansible/playbooks/setup_glinet_openwrt.yml` for a complete example.

---

## Security Notes

- **Change default passwords**: Admin password (web UI) and root password (SSH)
- **Disable WAN SSH**: Only allow SSH from LAN (default on GL.iNet)
- **Keep firmware updated**: Check for updates monthly via web UI
- **Use SSH keys**: Add your public key for passwordless Ansible access

---

## Next Steps

1. Complete initial router setup following steps above
2. Configure DHCP reservations (web UI or Ansible)
3. Run Ansible playbook for additional automation: `ansible-playbook -i inventory.yml playbooks/setup_glinet_openwrt.yml`
4. Document router MAC address in `docs/network_inventory.md`
