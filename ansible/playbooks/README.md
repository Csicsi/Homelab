# Ansible Playbooks

## Available Playbooks

### `bootstrap_glinet.yml`

**Target**: `glinet-router` (GL.iNet SF1200)

**Purpose**: Initial bootstrap - installs Python on router for Ansible to work

**What it does**:

- Checks if Python is already installed
- Updates opkg package list
- Installs Python 3 (required for Ansible modules)
- Verifies installation and tests Ansible connectivity

**Prerequisites**:

- Router accessible via SSH with RSA key
- Internet connection on router

**Usage**:

```bash
# Run immediately after SSH setup
ansible-playbook -i inventory.yml playbooks/bootstrap_glinet.yml
```

**Tags**: `bootstrap`, `test`

**Note**: This playbook uses `raw` commands (doesn't require Python), so it can run before Python is installed.

---

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
- **RSA SSH key generated and deployed** (router requires RSA, not ED25519):
  ```bash
  ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa_glinet -C "glinet-router"
  ssh-copy-id -i ~/.ssh/id_rsa_glinet -o HostKeyAlgorithms=+ssh-rsa -o PubkeyAcceptedKeyTypes=+ssh-rsa root@192.168.8.1
  ```
- SSH access working (test: `ansible -i inventory.yml glinet-router -m ping`)
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

### `setup_glinet_wireguard.yml`

**Target**: `glinet-router` (GL.iNet SF1200)

**Purpose**: Configure WireGuard VPN server on router for secure remote access to homelab

**What it does**:

- Installs WireGuard packages (wireguard-tools, kmod-wireguard)
- Generates server private/public key pair
- Creates WireGuard network interface (wg0)
- Configures firewall zones and rules
- Opens UDP port 51820 from WAN
- Allows VPN clients to access LAN

**Prerequisites**:

- Basic router setup completed (`setup_glinet_openwrt.yml`)
- Public IP or DDNS configured
- UDP port 51820 forwarded on LM1200 modem

**Usage**:

```bash
ansible-playbook -i inventory.yml playbooks/setup_glinet_wireguard.yml
```

**Tags**: `packages`, `wireguard`, `firewall`

**Post-setup**:

1. Note the server public key from playbook output
2. Create client configs via GL.iNet web UI or manually
3. Test VPN connection from remote location

**Note**: See `docs/glinet_setup.md` for WireGuard setup details and client configuration.

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
