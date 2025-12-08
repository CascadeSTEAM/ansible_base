# Quick Start Guide

This guide will help you get the Ansible SOHO infrastructure management system up and running quickly.

## Prerequisites

- Ubuntu 22.04+ server (control node)
- Root or sudo access
- Network access to managed systems

## Step 1: Install Ansible and Dependencies

```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install Ansible and dependencies
sudo apt install -y software-properties-common
sudo add-apt-repository --yes --update ppa:ansible/ansible
sudo apt install -y ansible python3-pip nmap

# Install Python requirements
pip3 install -r requirements.txt

# Install Ansible collections
ansible-galaxy collection install -r requirements.yml
```

## Step 2: Configure SSH Access

```bash
# Generate SSH key if you don't have one
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa

# Copy key to managed hosts (repeat for each host)
ssh-copy-id ansible@192.168.1.10
ssh-copy-id ansible@192.168.2.10
```

## Step 3: Configure Inventory

### Edit Network Ranges

Edit `inventory/dynamic/nmap_inventory.yml`:
```yaml
address: 
  - 192.168.1.0/24    # Your WiFi network
  - 192.168.2.0/24    # Your lab network
  - 10.0.0.0/24       # Your datacenter network
```

### Add Known Hosts

Edit `inventory/technitium_hosts.yml` and add your systems:
```yaml
lead-creative.local:
  ansible_host: 192.168.1.10
  role: area_lead
  area: creative
```

## Step 4: Test Connectivity

```bash
# Test SSH connection to a single host
ansible -i inventory/technitium_hosts.yml lead-creative.local -m ping

# List all discovered hosts
ansible-inventory --list

# Test all hosts
ansible all -m ping
```

## Step 5: Run Your First Playbook

```bash
# Detect systems on the network
ansible-playbook playbooks/detect_unknown_systems.yml

# Update Ubuntu systems
ansible-playbook playbooks/ubuntu_auto_update.yml

# Install applications
ansible-playbook playbooks/install_applications.yml

# Run everything
ansible-playbook playbooks/site.yml
```

## Step 6: Set Up Automation (Optional)

Create a cron job for regular maintenance:

```bash
# Edit crontab
crontab -e

# Add these lines:
# Detect unknown systems every hour
0 * * * * cd /home/ansible/ansible_base && ansible-playbook playbooks/detect_unknown_systems.yml

# Update systems daily at 2 AM
0 2 * * * cd /home/ansible/ansible_base && ansible-playbook playbooks/ubuntu_auto_update.yml
```

## Common Commands

```bash
# Check inventory
ansible-inventory --graph

# Ping all hosts
ansible all -m ping

# Get facts from a host
ansible lead-creative.local -m setup

# Run a single task
ansible lab_systems -m apt -a "name=vim state=present" --become

# Dry run (check mode)
ansible-playbook playbooks/site.yml --check

# Verbose output
ansible-playbook playbooks/site.yml -vvv
```

## Customization

### Change Update Schedule

Edit `inventory/group_vars/area_leads.yml`:
```yaml
update_schedule: "02:00"  # 2 AM
auto_reboot: true
auto_reboot_time: "02:30"
```

### Add Applications

Edit package lists in group_vars files:
```yaml
area_lead_packages:
  - libreoffice
  - gimp
  - your-package-here
```

### Configure Proxmox Integration

1. Set environment variables:
```bash
export PROXMOX_PASSWORD='your_password'
```

2. Update Proxmox URL in `inventory/dynamic/proxmox_inventory.yml`

## Troubleshooting

### "Host unreachable" errors
- Check SSH connectivity: `ssh ansible@<host>`
- Verify firewall rules
- Check network connectivity

### "Permission denied" errors
- Verify ansible user has sudo rights
- Check sudoers file: `/etc/sudoers.d/ansible`
- Ensure SSH key is copied: `ssh-copy-id ansible@<host>`

### No hosts discovered
- Verify network ranges in NMAP config
- Check that NMAP is installed: `which nmap`
- Test NMAP manually: `nmap -sn 192.168.1.0/24`

## Next Steps

1. Review and customize group variables in `inventory/group_vars/`
2. Add your specific hosts to `inventory/technitium_hosts.yml`
3. Test playbooks on non-production systems first
4. Set up monitoring and alerting
5. Configure backup schedules
6. Review security settings

## Getting Help

- Check logs: `./ansible.log` and `/var/log/ansible/`
- Run with verbose output: `-vvv`
- Review documentation: `README.md`
- Ansible docs: https://docs.ansible.com/

## Security Notes

- Always use SSH keys, never passwords
- Keep the ansible user password secure
- Don't commit `.env` files to git
- Review firewall rules regularly
- Monitor unknown system alerts
- Keep Ansible and collections updated
