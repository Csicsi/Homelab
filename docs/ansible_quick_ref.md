# Ansible Quick Reference

## Directory Structure

```
ansible/
├── ansible.cfg          # Ansible configuration
├── inventory.yml        # Host definitions (IPs, users, groups)
└── playbooks/          # Automation playbooks
    └── *.yml           # Individual playbook files
```

## Common Commands

```bash
# Test connectivity
ansible -i inventory.yml <host_or_group> -m ping

# Run playbook
ansible-playbook -i inventory.yml playbooks/<playbook>.yml

# Dry run (check mode)
ansible-playbook -i inventory.yml playbooks/<playbook>.yml --check

# Run specific tags only
ansible-playbook -i inventory.yml playbooks/<playbook>.yml --tags <tag_name>

# Limit to specific hosts
ansible-playbook -i inventory.yml playbooks/<playbook>.yml --limit <hostname>

# Verbose output
ansible-playbook -i inventory.yml playbooks/<playbook>.yml -v
```

## Inventory Groups

Use these group names with `ansible` or `--limit`:

- `servers` - x86 servers (ThinkPad, Asus)
- `pis` - All Raspberry Pis
- `staging` - Staging server only
- Or specific hostnames: `homelab-main`, `pi4-node1`, etc.

## Best Practices

**Before running playbooks**:

1. Update `inventory.yml` with correct usernames and IPs
2. Ensure SSH keys are deployed to target hosts
3. Test connectivity with `ansible <host> -m ping`

**When creating playbooks**:

- Use tags for logical task grouping
- Make playbooks idempotent (safe to run multiple times)
- Document what each playbook does (comments or README)
