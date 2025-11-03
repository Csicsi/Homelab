# Ansible Setup Guide

## Workstation Setup

### Install Ansible

```bash
# Ubuntu/Debian
sudo apt update
sudo apt install ansible -y

# Verify installation
ansible --version
```

### Generate SSH Key (if not already done)

```bash
# Generate ED25519 key (modern, secure)
ssh-keygen -t ed25519 -C "homelab-ansible"

# View public key
cat ~/.ssh/id_ed25519.pub
```

### Configure Inventory

Edit `ansible/inventory.yml` and update:

- Host IP addresses (from `docs/network_inventory.md`)
- Username for each host type (servers, Pis)

### Deploy SSH Keys to Hosts

```bash
# Copy key to target host
ssh-copy-id -i ~/.ssh/id_ed25519.pub user@host_ip

# Test passwordless SSH
ssh user@host_ip
```

Replace `user` and `host_ip` with actual values from inventory.

## Running Playbooks

### Basic Workflow

```bash
cd ~/Homelab/ansible

# Test connectivity to specific host group
ansible -i inventory.yml <group_name> -m ping

# Run a playbook
ansible-playbook -i inventory.yml playbooks/<playbook_name>.yml

# With sudo password prompt
ansible-playbook -i inventory.yml playbooks/<playbook_name>.yml --ask-become-pass
```

### Common Options

- `--ask-become-pass` - Prompt for sudo password
- `--tags <tag_name>` - Run only tasks with specific tags
- `--check` - Dry run (show what would change)
- `-v` / `-vv` / `-vvv` - Increase verbosity
- `--limit <hostname>` - Run on specific hosts only

## Troubleshooting

**SSH connection fails**: Verify IP address in inventory matches `network_inventory.md`

**Permission denied**: Ensure SSH key copied with `ssh-copy-id`

**Become password fails**: Enter correct sudo password for remote user

**Firewall blocks**: May need to temporarily allow SSH on remote host first
