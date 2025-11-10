# GL.iNet SF1200 Setup Guide

## Overview

The **GL.iNet GL-SF1200 (Slate)** is a dual-band travel router that ships with GL.iNet's customized OpenWrt-based firmware. It provides a polished web UI with full OpenWrt power underneath (UCI, opkg, SSH), making it ideal for homelab automation.

## Initial Setup

1. **Power on the router** and connect via WiFi or Ethernet
2. **Access web UI**: `http://192.168.8.1` (default) or `http://gl-sf1200.lan`
3. **Set admin password** on first boot
4. **Configure WAN**: Connect Ethernet cable from LM1200 to WAN port, verify internet connectivity
5. **SSH is enabled by default** - no web UI configuration needed

## SSH Access Setup

### Generate Compatible SSH Key

GL.iNet routers with older firmware may not accept ED25519 keys. Generate an RSA key:

```bash
# Generate RSA key for router
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa_glinet -C "glinet-router"

# Copy key to router (note the RSA algorithm options)
ssh-copy-id -i ~/.ssh/id_rsa_glinet -o HostKeyAlgorithms=+ssh-rsa -o PubkeyAcceptedKeyTypes=+ssh-rsa root@192.168.8.1
```

### Configure SSH Client

Create/edit `~/.ssh/config`:

```
Host 192.168.8.1 glinet glinet-router
    HostName 192.168.8.1
    User root
    IdentityFile ~/.ssh/id_rsa_glinet
    HostKeyAlgorithms +ssh-rsa
    PubkeyAcceptedKeyTypes +ssh-rsa
```

### Set Root Password

```bash
# First SSH connection (will prompt for new password)
ssh root@192.168.8.1

# Set a strong root password
# After this, SSH key authentication will work
```

### Test Passwordless Access

```bash
# Should work without password
ssh root@192.168.8.1
# or
ssh glinet-router
```

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
2. **Deploy SSH key**: `ssh-copy-id -i ~/.ssh/id_rsa_glinet -o HostKeyAlgorithms=+ssh-rsa -o PubkeyAcceptedKeyTypes=+ssh-rsa root@192.168.8.1`
3. **Ansible**: Configure everything else (network, DHCP, firewall, packages, etc.)

### Testing Ansible Connection

```bash
# Test connectivity
ansible -i ansible/inventory.yml glinet-router -m ping

# Expected: SUCCESS with "pong" response
# Note: SFTP/SCP warnings are normal on OpenWrt (uses dropbear, not OpenSSH)
```

See `ansible/playbooks/setup_glinet_openwrt.yml` for a complete example.

---

## WireGuard VPN Setup

The GL.iNet router serves as the WireGuard VPN server for secure remote access to the homelab.

### Manual Setup (Web UI)

1. **Install WireGuard**: Applications → Plug-ins → WireGuard Server (may be pre-installed)
2. **Configure Server**: VPN → WireGuard Server
   - Enable server
   - Note server public key and endpoint (your public IP or DDNS)
   - Default VPN subnet: `10.0.10.0/24` (or customize)
3. **Add Clients**:
   - Click "Add Client"
   - Generate client config
   - Download `.conf` file or scan QR code
4. **Port Forwarding**: Ensure UDP port 51820 is forwarded on LM1200 (if behind NAT)

### Ansible Setup

See `ansible/playbooks/setup_glinet_wireguard.yml` for automated WireGuard configuration:

- Installs WireGuard packages
- Generates server keys
- Configures interface and firewall
- Creates client configs

**Note**: Initial key generation should be done manually or via Ansible for security.

### Client Connection

```bash
# Linux/macOS
sudo wg-quick up /path/to/client.conf

# Or use WireGuard GUI client on Windows/mobile
```

### Accessing Homelab

Once connected to VPN:

- Router: `http://192.168.8.1`
- Switch: `http://192.168.8.2`
- Servers: `ssh user@192.168.8.10` (homelab-main)
- All homelab services accessible as if on local network

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
