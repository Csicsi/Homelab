# Ansible Playbooks

## Available Playbooks

### `setup_glinet_openwrt.yml`

**Target**: `glinet-router` (GL.iNet SF1200)

**Purpose**: Configure GL.iNet router network settings and DHCP reservations via UCI

**What it does**:

- Sets LAN IP address and netmask
- Configures DHCP server range and lease time
- Adds static DHCP reservations for all homelab devices
- Installs useful network tools (htop, tcpdump, iperf3)

**Prerequisites**:

- Router has stock GL.iNet firmware (default)
- Root password set manually via SSH
- SSH access enabled (System → Advanced Settings)
- **Copy vars file**: `cp ../vars/router_dhcp.yml.example ../vars/router_dhcp.yml`
- **Update MAC addresses** in `../vars/router_dhcp.yml` with actual device MACs

**Usage**:

```bash
# 1. Copy and edit vars file
cp ansible/vars/router_dhcp.yml.example ansible/vars/router_dhcp.yml
vim ansible/vars/router_dhcp.yml  # Add real MAC addresses

# 2. Run playbook
ansible-playbook -i inventory.yml playbooks/setup_glinet_openwrt.yml
```

**Tags**: `network`, `dhcp`, `packages`

**Note**: See `docs/glinet_setup.md` for initial router setup steps.

---

### `setup_main_server.yml`

**Target**: `homelab-main` (ThinkPad T440)

**Purpose**: Initial setup of main server with Docker, firewall, and laptop-specific configs

**What it does**:

- Updates all system packages
- Installs Docker and Docker Compose
- Configures laptop to stay running with lid closed
- Sets up SSH authorized keys
- Configures firewall for Docker and SSH

**Usage**:

```bash
ansible-playbook -i inventory.yml playbooks/setup_main_server.yml --ask-become-pass
```

**Tags**: `update`, `packages`, `docker`, `lid`, `ssh`, `firewall`

---

## Adding New Playbooks

When creating new playbooks, document them here with:

- Target hosts/groups
- Purpose (one-line summary)
- What it configures
- Usage example
- Available tags (if any)
