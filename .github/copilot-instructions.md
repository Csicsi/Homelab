# Homelab Infrastructure Project - AI Assistant Instructions

## Project Overview

This is a physical homelab infrastructure project focused on DevOps learning. The architecture spans mixed hardware (x86 laptops + ARM Raspberry Pis) managed entirely through Ansible automation. The project emphasizes reproducible infrastructure, configuration as code, and production-ready workflows on self-hosted hardware.

**Current Phase:** Foundation/Automation - Ansible-driven configuration management with planned progression to Kubernetes (k3s), CI/CD (Jenkins), and observability (Prometheus/Grafana).

## Architecture & Infrastructure

### Network Topology

- **IP Scheme:** 192.168.8.0/24 flat network with static DHCP reservations
- **Router:** GL.iNet SF1200 (OpenWrt-based) at 192.168.8.1 - managed via Ansible using UCI commands
- **Compute:** x86 servers (.10, .11) for heavy workloads; Raspberry Pis (.20-.22) for k3s and utilities
- All devices use static DHCP reservations defined in `ansible/vars/router_dhcp.yml`

### Host Groups (from `ansible/inventory.yml`)

- **routers:** GL.iNet SF1200 - requires RSA SSH keys (not ED25519), runs OpenWrt with Python3
- **servers:** Rocky Linux 9.x on ThinkPads/Asus laptops - Docker hosts, future Jenkins/monitoring nodes
- **pis:** Raspberry Pi OS 64-bit - designated for k3s cluster and lightweight services

### Key Design Decisions

- **Kubernetes Strategy:** k3s on Pi4s for orchestration learning; x86 hosts run Docker/Compose for resource-intensive services (Jenkins, databases)
- **Router Management:** GL.iNet uses UCI (Unified Configuration Interface) commands, not standard Linux networking
- **Laptop Configuration:** Servers ignore lid close events (`HandleLidSwitch=ignore`) for always-on operation

## Critical Workflows

### Ansible Execution Pattern

```bash
# Always run from ansible/ directory
cd ~/Homelab/ansible

# Test connectivity first
ansible -i inventory.yml <group> -m ping

# Run playbooks with explicit inventory
ansible-playbook -i inventory.yml playbooks/<playbook>.yml --ask-become-pass

# Use tags for targeted runs
ansible-playbook -i inventory.yml playbooks/setup_main_server.yml --tags docker
```

### GL.iNet Router Bootstrap Sequence

**Complete Pipeline (Recommended):**

```bash
export DUCKDNS_TOKEN="your_token"  # If using DDNS
ansible-playbook -i inventory.yml playbooks/setup_glinet_complete.yml
```

**Manual Step-by-Step:**

1. **First:** Run `bootstrap_glinet.yml` - installs Python3 via `raw` module (router ships without Python)
2. **Second:** Copy `ansible/vars/router_dhcp.yml.example` to `router_dhcp.yml` and populate real MAC addresses
3. **Third:** Run `setup_glinet_openwrt.yml` for network/DHCP configuration
4. **Optional:** Run `setup_glinet_ddns.yml` for DuckDNS dynamic DNS (needed for VPN with dynamic IP)
5. **Optional:** Run `setup_glinet_wireguard.yml` for WireGuard VPN server + client configs

**Critical:** Router SSH requires RSA keys with specific connection args (see `inventory.yml` ssh_common_args and `docs/ansible.md` for SSH setup).

### Adding New Hosts

1. Get MAC address: `ip link show` (Linux) or `cat /sys/class/net/eth0/address` (Pi)
2. Add DHCP reservation to `ansible/vars/router_dhcp.yml`
3. Update `ansible/inventory.yml` with hostname, IP, user, and become settings
4. Deploy SSH key: `ssh-copy-id -i ~/.ssh/id_ed25519.pub user@host_ip`
5. Test: `ansible -i inventory.yml <hostname> -m ping`

## Project-Specific Conventions

### Ansible Playbook Structure

- **All playbooks use tags** for granular task execution (e.g., `packages`, `docker`, `dhcp`, `firewall`)
- **Router tasks use `raw` or `command` modules** - GL.iNet/OpenWrt doesn't support standard Ansible network modules
- **Handlers pattern:** Laptop config tasks notify `Restart logind` handler for lid-close changes
- **External vars:** DHCP reservations live in separate `vars/router_dhcp.yml` (excluded from git via .example pattern)

### Documentation Standards

- **Primary sources:** `docs/` directory contains setup guides, not just README
  - `docs/ansible.md` - Complete Ansible setup, commands, and workflows
  - `docs/network_setup.md` - Network topology, router setup, and DHCP configuration
  - `docs/network_inventory.md` - Source of truth for IPs and MACs
- **Playbook README:** `ansible/playbooks/README.md` documents every playbook with prerequisites, usage, and tags

### File Naming Patterns

- Playbooks: `setup_<target>_<purpose>.yml` (e.g., `setup_glinet_wireguard.yml`)
- Bootstrap playbooks use `bootstrap_` prefix for initial Python/dependency installation
- Vars files use `.yml.example` for templates with sensitive data (real `.yml` is gitignored)

## Technology-Specific Guidance

### OpenWrt/GL.iNet Router Management

- Use `uci set/add/delete` commands, then `uci commit <service>` and restart service
- Example: Adding DHCP reservation requires `uci add dhcp host`, set name/mac/ip, commit dhcp, restart dnsmasq
- Router reboots needed for some network changes: `ansible.builtin.reboot` with appropriate wait_for tasks

### Rocky Linux (RHEL-based) Specifics

- Package manager: `ansible.builtin.dnf` (not apt)
- Docker repo: Uses CentOS repos (`docker-ce.repo`) - see `setup_main_server.yml`
- Firewall: `firewalld` with `ansible.posix.firewalld` module - always set `permanent: true` and `immediate: true`

### Multi-Architecture Awareness

- Pi playbooks may need ARM-specific package names or container images
- Future k3s setup requires `--disable servicelb` on ARM to avoid conflicts with MetalLB

## Integration Points

### Planned Services (not yet implemented)

- **Jenkins:** Will run on `homelab-main` (.10) with GitHub webhook integration
- **k3s:** Multi-node cluster across pi4-node1 and pi4-node2 with shared storage via NFS or Longhorn
- **Prometheus/Grafana:** Centralized monitoring on x86 host, node exporters on all devices

### Implemented Services

- **WireGuard VPN:** Server on router with automated client config generation (playbook: `setup_glinet_wireguard.yml`)
- **DuckDNS:** Dynamic DNS for remote access with changing public IP (playbook: `setup_glinet_ddns.yml`)

### External Dependencies

- **SSH keys:** Control machine must have ED25519 keys for Linux hosts, RSA keys for router
- **Internet access:** Required during playbook runs for package installation (opkg, dnf)
- **DuckDNS token:** Required for DDNS setup - get free token at https://www.duckdns.org
- **Port forwarding:** WireGuard VPN requires UDP 51820 forwarded from ISP modem to router (or configure on LM1200)

## Common Pitfalls

1. **Running playbooks from wrong directory** - Must be in `ansible/` or provide absolute inventory path
2. **Using ED25519 keys with GL.iNet** - Router requires RSA; update `~/.ssh/config` with `HostKeyAlgorithms +ssh-rsa`
3. **Forgetting --ask-become-pass** - Required for privilege escalation on servers/pis
4. **Skipping bootstrap on router** - `bootstrap_glinet.yml` must run before any other router playbooks
5. **Not committing UCI changes** - OpenWrt requires explicit `uci commit` after configuration changes
6. **Docker group changes** - User must log out/in after being added to docker group (or use `newgrp docker`)

## When Modifying This Codebase

- **New playbooks:** Document in `ansible/playbooks/README.md` with usage examples and tags
- **Network changes:** Update both `ansible/inventory.yml` AND `docs/network_inventory.md`
- **New services:** Check resource constraints - Pi4s have only 4GB RAM, reserve heavy workloads for x86
- **Router config:** Always test changes with `--check` first; router reboots can disrupt network access
- **SSH keys:** Document key type requirements in playbook prerequisites (RSA for router, ED25519 for Linux)
