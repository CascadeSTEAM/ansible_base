# Ansible SOHO Infrastructure Management

A comprehensive Ansible-based infrastructure management system designed for Small Office/Home Office (SOHO) environments, specifically tailored for makerspaces with public WiFi, lab systems, and secure datacenter infrastructure.

## Overview

This Ansible playbook system manages:
- **Dynamic inventory discovery** using NMAP, Proxmoxer, Constructed, and Technitium DNS
- **Security monitoring** for unknown systems on the network
- **Automatic updates** for Ubuntu Linux desktop systems
- **Application management** with role-based software installation
- **Lab systems** for public use
- **Area lead workstations** for makerspace managers
- **Datacenter infrastructure** with secure access controls

## Architecture

```
ansible_base/
├── ansible.cfg                 # Main Ansible configuration
├── inventory/
│   ├── dynamic/               # Dynamic inventory plugin configurations
│   │   ├── nmap_inventory.yml
│   │   ├── proxmox_inventory.yml
│   │   └── constructed_inventory.yml
│   ├── technitium_hosts.yml   # DNS-managed static hosts
│   ├── group_vars/            # Group-specific variables
│   │   ├── all.yml
│   │   ├── area_leads.yml
│   │   ├── lab_systems.yml
│   │   ├── datacenter.yml
│   │   └── wifi_clients.yml
│   └── host_vars/             # Host-specific variables
├── playbooks/
│   ├── site.yml               # Master playbook
│   ├── detect_unknown_systems.yml
│   ├── ubuntu_auto_update.yml
│   ├── install_applications.yml
│   ├── refresh_inventory.yml
│   └── templates/             # Jinja2 templates
└── roles/
    ├── security_monitoring/   # Security and monitoring role
    ├── system_updates/        # System update management role
    └── app_management/        # Application installation role
```

## Prerequisites

### Control Node (Ansible Server)

1. **Operating System**: Ubuntu 22.04 LTS or later
2. **Ansible**: Version 2.14 or later
3. **Python**: Python 3.8 or later
4. **Required Python packages**:
   ```bash
   pip3 install ansible paramiko proxmoxer requests
   ```

5. **Required Ansible Collections**:
   ```bash
   ansible-galaxy collection install community.general
   ansible-galaxy collection install ansible.posix
   ```

### Managed Nodes

1. **Ubuntu 20.04/22.04/24.04 LTS** for desktop systems
2. **SSH access** with key-based authentication
3. **sudo privileges** for the ansible user
4. **Python 3** installed

## Installation

### 1. Clone the Repository

```bash
git clone https://github.com/CascadeSTEAM/ansible_base.git
cd ansible_base
```

### 2. Configure Ansible User

Create an ansible user on all managed systems:

```bash
# On each managed system
sudo adduser ansible
sudo usermod -aG sudo ansible
echo "ansible ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/ansible
```

### 3. Set Up SSH Keys

```bash
# On the control node
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa
ssh-copy-id ansible@<target-host>
```

### 4. Configure Inventory

Edit inventory files to match your network:

**NMAP Inventory** (`inventory/dynamic/nmap_inventory.yml`):
- Update network ranges in the `address` section
- Adjust exclusions for routers, APs, etc.

**Proxmox Inventory** (`inventory/dynamic/proxmox_inventory.yml`):
- Set Proxmox server URL
- Configure authentication (use environment variables)
- Update tag mappings

**Technitium Hosts** (`inventory/technitium_hosts.yml`):
- Add DNS-registered systems
- Update IP addresses and hostnames

### 5. Configure Group Variables

Edit files in `inventory/group_vars/` to customize:
- Software packages
- Update schedules
- Security settings
- Backup configurations

## Usage

### Running the Master Playbook

Execute all management tasks:

```bash
ansible-playbook playbooks/site.yml
```

### Individual Playbooks

**Detect Unknown Systems:**
```bash
ansible-playbook playbooks/detect_unknown_systems.yml
```

**Update Ubuntu Systems:**
```bash
ansible-playbook playbooks/ubuntu_auto_update.yml
```

**Install Applications:**
```bash
ansible-playbook playbooks/install_applications.yml
```

**Refresh Dynamic Inventory:**
```bash
ansible-playbook playbooks/refresh_inventory.yml
```

### Inventory Commands

**List all hosts:**
```bash
ansible-inventory --list
```

**View specific group:**
```bash
ansible-inventory --graph area_leads
```

**Test connectivity:**
```bash
ansible all -m ping
```

## Network Topology

The system manages three primary network segments:

1. **WiFi Network (192.168.1.0/24)**
   - Public WiFi clients
   - Area lead workstations
   - General member access

2. **Lab Network (192.168.2.0/24)**
   - Public-use lab stations
   - Development systems
   - Educational workstations

3. **Datacenter Network (10.0.0.0/24)**
   - Proxmox virtualization hosts
   - NAS storage
   - Core infrastructure services

## Security Features

### Unknown System Detection

The system continuously monitors for unauthorized devices:
- Scans network using NMAP
- Compares against known inventory
- Generates alerts for unknown systems
- Logs detection events

### Automatic Updates

Ubuntu systems are kept secure with:
- Daily security updates
- Configurable reboot schedules
- Role-based update policies
- Update logging and reporting

### Firewall Management

UFW firewall configuration:
- Deny-by-default policy
- Role-based service allowlists
- Automated rule management

### Fail2Ban Integration

Intrusion prevention:
- SSH brute-force protection
- Configurable ban times
- Email notifications

## Application Management

### Area Lead Systems

Productivity and management software:
- LibreOffice suite
- GIMP, Inkscape (graphics)
- Communication tools (Slack, Thunderbird)
- Remote desktop capabilities

### Lab Systems

Maker-focused applications:
- Development tools (VS Code, Git, Node.js, Python)
- CAD software (KiCAD, FreeCAD, Blender)
- 3D printing tools (Cura, PrusaSlicer)
- Electronics tools (Arduino, Fritzing)
- Media production (OBS Studio, Audacity)

## Automation

### Cron Jobs

Set up regular execution:

```bash
# Run unknown system detection every hour
0 * * * * cd /home/ansible/ansible_base && ansible-playbook playbooks/detect_unknown_systems.yml

# Run updates daily at 2 AM
0 2 * * * cd /home/ansible/ansible_base && ansible-playbook playbooks/ubuntu_auto_update.yml

# Weekly full infrastructure check (Sunday 3 AM)
0 3 * * 0 cd /home/ansible/ansible_base && ansible-playbook playbooks/site.yml
```

## Environment Variables

For Proxmox integration:

```bash
export PROXMOX_PASSWORD='your_proxmox_password'
```

For Technitium DNS integration (if using API):

```bash
export TECHNITIUM_SERVER='dns.example.com'
export TECHNITIUM_API_TOKEN='your_api_token'
```

## Logging

Logs are stored in:
- `/var/log/ansible/` - Main Ansible logs
- `./ansible.log` - Local execution log
- `/var/log/ansible/unknown_systems.log` - Unknown system detection log
- `/var/log/ansible/reboot_required.log` - Systems requiring reboot

## Customization

### Adding New Host Groups

1. Create group variables in `inventory/group_vars/<group_name>.yml`
2. Update constructed inventory rules in `inventory/dynamic/constructed_inventory.yml`
3. Add group-specific tasks to playbooks as needed

### Adding Applications

Edit the appropriate group_vars file:
- `area_leads.yml` - Area lead packages
- `lab_systems.yml` - Lab system packages
- `all.yml` - Common packages

### Modifying Update Schedules

Edit update-related variables in group_vars:
```yaml
update_schedule: "02:00"
auto_reboot: true
auto_reboot_time: "02:30"
```

## Troubleshooting

### Inventory Not Updating

```bash
# Clear fact cache
rm -rf /tmp/ansible_facts/*

# Refresh inventory manually
ansible-inventory --list > /tmp/inventory_debug.json
```

### Connection Issues

```bash
# Test SSH connectivity
ansible all -m ping -vvv

# Check SSH configuration
ssh -vvv ansible@<target-host>
```

### NMAP Discovery Issues

- Ensure NMAP is installed: `sudo apt install nmap`
- Verify network ranges in configuration
- Check firewall rules allow ICMP

### Proxmox Authentication

- Verify credentials and permissions
- Check SSL certificate validation settings
- Test API access: `curl -k https://proxmox-server:8006/api2/json/version`

## Best Practices

1. **Test in staging** - Always test playbooks on non-production systems first
2. **Use tags** - Add Proxmox tags for better inventory organization
3. **Regular backups** - Back up inventory configurations
4. **Monitor logs** - Review unknown system logs regularly
5. **Update schedules** - Stagger update times across groups
6. **Security audits** - Regularly review firewall and fail2ban logs

## Support and Contributing

For issues, questions, or contributions:
- GitHub Issues: https://github.com/CascadeSTEAM/ansible_base/issues
- Documentation: This README
- Ansible Documentation: https://docs.ansible.com/

## License

See LICENSE file for details.

## Acknowledgments

Built for makerspace infrastructure management with public access, lab systems, and secure datacenter integration.
