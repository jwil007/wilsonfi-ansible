# wilsonfi-ansible

Ansible for the wilsonfi lab. NetBox is the source of truth and the inventory;
Proxmox hosts are driven through their API.

## Layout

| Path | What |
|---|---|
| `inventory/netbox.yml` | Dynamic inventory: NetBox VMs/devices tagged `ansible` and `active` |
| `inventory/hosts.yml` | Static `unregistered` group for discover-first onboarding |
| `inventory/group_vars/all/` | `vars.yml`, `proxmox.yml` (one entry per Proxmox host), `vault.yml` (encrypted) |
| `roles/` | Reusable "how to configure X": bootstrap, users, ssh_hardening, motd, patch, netbox_sync |
| `playbooks/` | Orchestration: what runs where, in what order |

## Common runs

```bash
ansible-playbook playbooks/provision.yml -e target=<vm>      # NetBox Planned → running, configured, Active LXC
ansible-playbook playbooks/decommission.yml -e target=<vm>   # NetBox Decommissioning → destroyed, record removed
ansible-playbook playbooks/baseline.yml                      # standard config; --check --diff = drift check
ansible-playbook playbooks/baseline.yml --tags motd          # one role only
ansible-playbook playbooks/patch.yml                         # rings: sandbox → utility → core (serial)
ansible-playbook playbooks/compliance.yml                    # report-only; fails if anything is out of line
```

## Setup on a control node

```bash
pipx install ansible-core && pipx inject ansible-core -r requirements.txt
ansible-galaxy collection install -r requirements.yml
pipx install ansible-lint      # then: ansible-lint
```
Vault password goes in `~/.vault_pass` (never committed).
