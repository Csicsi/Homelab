# Ansible Guide

## Overview

Ansible automates all homelab configuration management - from initial system setup to service deployment. All nodes (servers, Pis, router) are managed through Ansible playbooks with idempotent, version-controlled infrastructure.

---

## Directory Structure

```
ansible/
├── ansible.cfg          # Ansible configuration
├── inventory.yml        # Host definitions (IPs, users, groups)
├── vars/
│   └── router_dhcp.yml  # DHCP reservations (copy from .example)
└── playbooks/
    ├── README.md                    # Playbook documentation
    ├── bootstrap_glinet.yml         # Router Python installation
    ├── setup_glinet_openwrt.yml     # Router network/DHCP config
    ├── setup_glinet_wireguard.yml   # VPN setup
    ├── setup_main_server.yml        # Main server setup
    └── setup_glinet_ddns.yml        # Dynamic DNS
```

---

## Initial Setup

### 1. Install Ansible

Ubuntu/Debian:

```bash
sudo apt update
sudo apt install ansible -y
```

Verify installation:

```bash
ansible --version
```

### 2. Generate SSH Keys

For Linux hosts (servers, Pis) - use ED25519:

```bash
ssh-keygen -t ed25519 -C "homelab-ansible"
```

For GL.iNet router - use RSA (compatibility requirement):

```bash
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa_glinet -C "glinet-router"
```

View public keys:

```bash
cat ~/.ssh/id_ed25519.pub
cat ~/.ssh/id_rsa_glinet.pub
```

### 3. Configure SSH for Router

Create/edit `~/.ssh/config`:

```
Host 192.168.8.1 glinet glinet-router
    HostName 192.168.8.1
    User root
    IdentityFile ~/.ssh/id_rsa_glinet
    HostKeyAlgorithms +ssh-rsa
    PubkeyAcceptedKeyTypes +ssh-rsa
```

### 4. Deploy SSH Keys

Linux hosts:

```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub user@host_ip
```

Router (note the algorithm options):

```bash
ssh-copy-id -i ~/.ssh/id_rsa_glinet -o HostKeyAlgorithms=+ssh-rsa -o PubkeyAcceptedKeyTypes=+ssh-rsa root@192.168.8.1
```

Test passwordless access:

```bash
ssh user@host_ip
ssh glinet-router
```

### 5. Verify Inventory

Edit `ansible/inventory.yml` and confirm:

- Host IP addresses match `docs/network_inventory.md`
- Usernames are correct for each host
- SSH key paths are correct

---

## Running Playbooks

### Basic Workflow

Always run from ansible/ directory:

```bash
cd ~/Homelab/ansible
```

Test connectivity:

```bash
ansible -i inventory.yml <group_or_host> -m ping
```

Run a playbook:

```bash
ansible-playbook -i inventory.yml playbooks/<playbook>.yml
```

With sudo password prompt (required for servers/pis):

```bash
ansible-playbook -i inventory.yml playbooks/<playbook>.yml --ask-become-pass
```

### Router Bootstrap Sequence

**IMPORTANT:** Router requires Python installation before other playbooks can run.

Bootstrap - install Python on router (uses raw commands):

```bash
ansible-playbook -i inventory.yml playbooks/bootstrap_glinet.yml
```

Copy and edit DHCP vars file:

```bash
cp vars/router_dhcp.yml.example vars/router_dhcp.yml
vim vars/router_dhcp.yml
```

Add real MAC addresses to the vars file, then configure router network and DHCP:

```bash
ansible-playbook -i inventory.yml playbooks/setup_glinet_openwrt.yml
```

### Common Options

Dry run (show what would change):

```bash
ansible-playbook -i inventory.yml playbooks/<playbook>.yml --check
```

Run specific tags only:

```bash
ansible-playbook -i inventory.yml playbooks/<playbook>.yml --tags docker,firewall
```

Limit to specific hosts:

```bash
ansible-playbook -i inventory.yml playbooks/<playbook>.yml --limit homelab-main
```

Verbose output (add more v's for detail):

```bash
ansible-playbook -i inventory.yml playbooks/<playbook>.yml -vvv
```

---

## Inventory Groups

Use these group names with `ansible` commands or `--limit`:

- **`routers`** - GL.iNet SF1200 at 192.168.8.1
- **`servers`** - x86 servers (ThinkPad, Asus) at .10, .11
- **`pis`** - All Raspberry Pis at .20-.22
- **`staging`** - Staging server only (.11)
- **Specific hosts:** `homelab-main`, `pi4-node1`, `glinet-router`, etc.

---

## Playbook Reference

See `ansible/playbooks/README.md` for detailed documentation of each playbook.

### Key Playbooks

**`bootstrap_glinet.yml`**

- Installs Python3 on router (required first)
- Uses `raw` commands (doesn't need Python)
- Run once before any other router playbooks

**`setup_glinet_openwrt.yml`**

- Configures router LAN IP and DHCP
- Adds static DHCP reservations for all hosts
- Requires `vars/router_dhcp.yml` with real MAC addresses

**`setup_main_server.yml`**

- System updates and package installation
- Docker and Docker Compose setup
- Laptop lid-close configuration
- Firewall setup for Docker and SSH
- Requires `--ask-become-pass`

**`setup_glinet_wireguard.yml`**

- WireGuard VPN server on router
- Generates keys and configures firewall
- Opens UDP port 51820

---

## Best Practices

### Before Running Playbooks

1. **Update inventory**: Ensure IPs and usernames are correct
2. **Deploy SSH keys**: Test passwordless SSH to all targets
3. **Test connectivity**: `ansible -i inventory.yml <host> -m ping`
4. **Check vars files**: Copy `.example` files and fill in real values

### When Creating Playbooks

- **Use tags**: Group related tasks (e.g., `packages`, `docker`, `firewall`)
- **Be idempotent**: Playbooks should be safe to run multiple times
- **Document**: Add playbook details to `playbooks/README.md`
- **Test first**: Use `--check` mode for dry runs
- **Handle errors**: Use `ignore_errors` or `failed_when` appropriately

### Common Patterns

**Router tasks** - Use `command` or `raw` modules for UCI:

```yaml
- name: Set DHCP range
  ansible.builtin.command:
    cmd: uci set dhcp.lan.start='100'
```

**Rocky Linux packages** - Use `dnf` not `apt`:

```yaml
- name: Install packages
  ansible.builtin.dnf:
    name: [vim, git, htop]
    state: present
```

**Firewall rules** - Always use `permanent: true` and `immediate: true`:

```yaml
- name: Allow SSH
  ansible.posix.firewalld:
    service: ssh
    permanent: true
    immediate: true
    state: enabled
```

---

## Troubleshooting

**SSH connection fails**

- Verify IP in inventory matches `docs/network_inventory.md`
- Test manual SSH: `ssh user@host_ip`
- Check SSH key deployed: `ssh-copy-id -i ~/.ssh/id_ed25519.pub user@host_ip`

**Permission denied (publickey)**

- Ensure SSH key copied to remote host
- For router: Use RSA key with proper algorithms in ssh_config

**Become password fails**

- Enter correct sudo password for remote user
- Verify user has sudo privileges: `sudo -l`

**Router playbooks fail**

- Run `bootstrap_glinet.yml` first to install Python
- Verify router accessible: `ansible -i inventory.yml glinet-router -m ping`

**Module not found**

- Install collection: `ansible-galaxy collection install ansible.posix`
- Check `ansible.cfg` for correct paths

---

## Adding New Hosts

1. **Get MAC address**:

   Linux:

   ```bash
   ip link show
   ```

   Raspberry Pi:

   ```bash
   cat /sys/class/net/eth0/address
   ```

2. **Add DHCP reservation**: Edit `ansible/vars/router_dhcp.yml`

   ```yaml
   - name: "new-host"
     mac: "XX:XX:XX:XX:XX:XX"
     ip: "192.168.8.XX"
   ```

3. **Update inventory**: Add to `ansible/inventory.yml`

   ```yaml
   servers:
     hosts:
       new-host:
         ansible_host: 192.168.8.XX
         ansible_user: username
         ansible_become: true
   ```

4. **Deploy DHCP changes**:

   ```bash
   ansible-playbook -i inventory.yml playbooks/setup_glinet_openwrt.yml --tags dhcp
   ```

5. **Deploy SSH key and test**:

   ```bash
   ssh-copy-id -i ~/.ssh/id_ed25519.pub username@192.168.8.XX
   ansible -i inventory.yml new-host -m ping
   ```

---

## Next Steps

- Review playbook documentation: `ansible/playbooks/README.md`
- Check network configuration: `docs/network_setup.md`
- See hardware specs: `docs/hardware.md`
- View software stack: `docs/software_stack.md`
