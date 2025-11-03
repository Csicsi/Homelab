# Ansible Playbooks

## Available Playbooks

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
