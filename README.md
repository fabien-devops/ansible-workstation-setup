# Ansible Workstation Setup

> Automated workstation and server configuration using Ansible for consistent development environments.

[![Ansible](https://img.shields.io/badge/Ansible-EE0000?style=flat&logo=ansible&logoColor=white)](https://www.ansible.com/)
[![Linux](https://img.shields.io/badge/Linux-FCC624?style=flat&logo=linux&logoColor=black)](https://www.linux.org/)
[![Ubuntu](https://img.shields.io/badge/Ubuntu-E95420?style=flat&logo=ubuntu&logoColor=white)](https://ubuntu.com/)

## Overview

This Ansible playbook automates the setup of development workstations and web servers. It handles user creation, system updates, essential software installation, custom MOTD (Message of the Day), hostname configuration, and timezone settings—ensuring a consistent and ready-to-use environment across multiple machines.

## Features

- 👤 **User Management**: Automated user account creation with sudo privileges
- 📦 **Package Management**: System updates and essential development tools installation
- 🎨 **Custom MOTD**: Personalized welcome message on SSH login
- 🖥️ **Hostname Configuration**: Automatic hostname setup based on inventory
- 🌍 **Timezone Configuration**: Sets system timezone to UTC
- ⚙️ **Idempotent**: Safe to run multiple times without side effects
- 🔐 **SSH Configuration**: Custom port and key-based authentication support

## Prerequisites

### Control Node (Your Machine)

```bash
# Ansible installed
sudo apt update
sudo apt install ansible

# Verify installation
ansible --version
```

### Target Hosts

- SSH access with sudo privileges
- Python 3 installed (usually pre-installed on Ubuntu/Debian)
- Network connectivity between control node and target hosts

## Quick Start

### 1. Clone the Repository

```bash
git clone https://github.com/fabien-devops/ansible-workstation-setup.git
cd ansible-workstation-setup
```

### 2. Configure Your Inventory

Edit `inventory.ini` with your server details:

```ini
[webservers]
webserver1 ansible_host=92.112.23.4
webserver2 ansible_host=192.168.1.100
# Add more servers as needed
```

### 3. Configure Variables

Edit `group_vars/webservers/vars.yml` to customize:

```yaml
user: bossman                    # Username to create
user_group: sudo                 # Group membership
shell: /bin/bash                 # Default shell
ansible_user: user               # SSH user for connection
ansible_ssh_port: 2222          # Custom SSH port
ansible_ssh_private_key_file: ~/.ssh/id_ed25519
```

### 4. Run the Playbook

```bash
# Run with default settings
ansible-playbook -i inventory.ini playbook.yml

# Run with custom user
ansible-playbook -i inventory.ini playbook.yml --ask-become-pass

# Dry run (check mode)
ansible-playbook -i inventory.ini playbook.yml --check
```

## Project Structure

```
ansible-workstation-setup/
├── inventory.ini              # Target hosts definition
├── playbook.yml              # Main Ansible playbook
├── group_vars/
│   └── webservers/
│       └── vars.yml          # Variables for webservers group
├── motd                      # Custom MOTD script
└── README.md                 # This file
```

## Configuration Details

### Inventory Configuration

The `inventory.ini` file defines your target hosts:

```ini
[webservers]
webserver1 ansible_host=92.112.23.4
webserver2 ansible_host=10.0.0.5
```

**Tips:**
- Use descriptive hostnames (e.g., `prod-web01`, `dev-workstation`)
- Group similar hosts together
- Use IP addresses or DNS names for `ansible_host`

### Variables Configuration

Located in `group_vars/webservers/vars.yml`:

| Variable | Description | Example |
|----------|-------------|---------|
| `user` | Username to create | `bossman` |
| `user_group` | Primary group | `sudo` |
| `shell` | Default shell | `/bin/bash` |
| `ansible_user` | SSH connection user | `user` |
| `ansible_ssh_port` | SSH port | `2222` |
| `ansible_ssh_private_key_file` | SSH key path | `~/.ssh/id_ed25519` |
| `ansible_ssh_common_args` | SSH options | `-o StrictHostKeyChecking=no` |

### Custom MOTD Script

The `motd` file creates a custom welcome message:

```bash
#!/bin/bash
echo "======================================="
echo " Welcome to $(hostname) "
echo " User: $(whoami) | Date: $(date '+%Y-%m-%d')"
echo "======================================="
echo ""
echo "Remember: Keep your server updated and secure!"
echo ""
```

**Location on target:** `/etc/update-motd.d/99-custom`

## What Gets Installed

### Essential Development Tools

- **git** - Version control system
- **curl** - Data transfer tool
- **wget** - Network downloader
- **htop** - Interactive process viewer
- **tree** - Directory structure visualizer
- **vim** - Advanced text editor
- **nano** - Simple text editor
- **nodejs** - JavaScript runtime

### System Operations

1. **APT Update & Upgrade**: Full system update with distribution upgrade
2. **Autoremove**: Cleanup unnecessary packages
3. **User Creation**: New user with sudo privileges
4. **MOTD Setup**: Custom welcome message
5. **Hostname Configuration**: Set system hostname
6. **Timezone Setup**: Configure UTC timezone

## Playbook Breakdown

### Task 1: Create User Account
```yaml
- name: Create a user account
  user:
    name: "{{ user }}"
    groups: "{{ user_group }}"
    shell: "{{ shell }}"
    create_home: yes
```
Creates a new user with sudo access and home directory.

### Task 2: System Update
```yaml
- name: Update apt
  apt:
    update_cache: yes
    upgrade: dist
    autoremove: yes
```
Updates package lists, performs distribution upgrade, and removes unused packages.

### Task 3: Software Installation
```yaml
- name: Install Software
  apt:
    name: [git, curl, wget, htop, tree, vim, nano, nodejs]
    state: present
```
Installs essential development and system administration tools.

### Task 4: Custom MOTD
```yaml
- name: Set MOTD
  ansible.builtin.copy:
    src: motd
    dest: /etc/update-motd.d/99-custom
    owner: root
    group: root
    mode: '0755'
```
Copies and sets up custom Message of the Day script.

### Task 5: Hostname Configuration
```yaml
- name: Set the system hostname
  ansible.builtin.hostname:
    name: "{{ inventory_hostname }}"
  when: ansible_fqdn != inventory_hostname
```
Configures system hostname based on inventory definition.

### Task 6: Timezone Setup
```yaml
- name: Set timezone UTC
  community.general.timezone:
    name: Etc/UTC
```
Sets system timezone to UTC for consistency.

## Usage Examples

### Single Host Setup

```bash
# Target specific host
ansible-playbook -i inventory.ini playbook.yml --limit webserver1
```

### Multiple Hosts

```bash
# Run on all webservers
ansible-playbook -i inventory.ini playbook.yml

# Run on specific hosts
ansible-playbook -i inventory.ini playbook.yml --limit "webserver1,webserver2"
```

### With Different User

```bash
# Use different SSH user
ansible-playbook -i inventory.ini playbook.yml -u ubuntu
```

### Verbose Output

```bash
# Debug mode
ansible-playbook -i inventory.ini playbook.yml -vvv
```

### Check Mode (Dry Run)

```bash
# See what would change without making changes
ansible-playbook -i inventory.ini playbook.yml --check --diff
```

## Advanced Configuration

### Custom SSH Configuration

For non-standard SSH setups, modify `group_vars/webservers/vars.yml`:

```yaml
# Custom SSH port
ansible_ssh_port: 2222

# Specific SSH key
ansible_ssh_private_key_file: ~/.ssh/custom_key

# Additional SSH options
ansible_ssh_common_args: '-o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null'
```

### Multiple Environment Support

Create separate variable files for different environments:

```
group_vars/
├── webservers/
│   ├── vars.yml          # Common variables
│   ├── dev.yml           # Development specific
│   └── prod.yml          # Production specific
```

Run with specific environment:
```bash
ansible-playbook -i inventory.ini playbook.yml -e @group_vars/webservers/prod.yml
```

### Adding More Software

Edit `playbook.yml` to add packages:

```yaml
- name: Install Software
  apt:
    name:
      - git
      - docker.io      # Add Docker
      - python3-pip    # Add pip
      - postgresql     # Add PostgreSQL
    state: present
```

## SSH Configuration

### Generate SSH Key

```bash
# Generate ED25519 key (recommended)
ssh-keygen -t ed25519 -C "your_email@example.com"

# Copy to target host
ssh-copy-id -i ~/.ssh/id_ed25519.pub user@92.112.23.4 -p 2222
```

### Test Connection

```bash
# Test SSH connectivity
ansible webservers -i inventory.ini -m ping

# Expected output:
# webserver1 | SUCCESS => {
#     "changed": false,
#     "ping": "pong"
# }
```

## Troubleshooting

### Connection Refused

**Problem:** Cannot connect to target host

```bash
# Solution 1: Verify SSH connectivity
ssh user@92.112.23.4 -p 2222

# Solution 2: Check firewall
sudo ufw status
sudo ufw allow 2222/tcp

# Solution 3: Verify SSH service
sudo systemctl status sshd
```

### Permission Denied

**Problem:** User doesn't have sudo privileges

```bash
# Solution: Add user to sudo group on target
ssh user@host
sudo usermod -aG sudo username
```

### Module Not Found

**Problem:** `community.general.timezone` module not found

```bash
# Solution: Install Ansible community collection
ansible-galaxy collection install community.general
```

### Host Key Verification Failed

**Problem:** SSH host key verification fails

```bash
# Solution 1: Remove old host key
ssh-keygen -R 92.112.23.4

# Solution 2: Use StrictHostKeyChecking=no (less secure)
# Already configured in vars.yml
```

## Security Considerations

### Best Practices

1. ✅ **Use SSH Keys**: Never use password authentication
2. ✅ **Custom SSH Port**: Change from default port 22
3. ✅ **Limit Sudo Access**: Only give sudo to trusted users
4. ✅ **Keep Systems Updated**: Regular security updates
5. ✅ **Use Ansible Vault**: Encrypt sensitive variables
6. ✅ **Firewall Rules**: Configure UFW or iptables

### Using Ansible Vault

Encrypt sensitive data:

```bash
# Create encrypted file
ansible-vault create group_vars/webservers/vault.yml

# Edit encrypted file
ansible-vault edit group_vars/webservers/vault.yml

# Run playbook with vault
ansible-playbook -i inventory.ini playbook.yml --ask-vault-pass
```

## Customization Ideas

### Add Docker Installation

```yaml
- name: Install Docker
  apt:
    name:
      - docker.io
      - docker-compose
    state: present

- name: Add user to docker group
  user:
    name: "{{ user }}"
    groups: docker
    append: yes
```

### Configure Firewall

```yaml
- name: Install UFW
  apt:
    name: ufw
    state: present

- name: Allow SSH
  community.general.ufw:
    rule: allow
    port: "{{ ansible_ssh_port }}"
    proto: tcp

- name: Enable UFW
  community.general.ufw:
    state: enabled
```

### Add Monitoring Tools

```yaml
- name: Install monitoring tools
  apt:
    name:
      - prometheus-node-exporter
      - netdata
    state: present
```

## Verification

After running the playbook, verify the setup:

```bash
# SSH into the server
ssh bossman@92.112.23.4 -p 2222

# Check installed packages
dpkg -l | grep -E "git|curl|nodejs"

# Verify user
id bossman

# Check MOTD
cat /etc/update-motd.d/99-custom

# Verify hostname
hostnamectl

# Check timezone
timedatectl
```

## Performance Tips

### Parallel Execution

```bash
# Run on multiple hosts in parallel (default is 5)
ansible-playbook -i inventory.ini playbook.yml -f 10
```

### Limit Output

```bash
# Show only changes
ansible-playbook -i inventory.ini playbook.yml --diff
```

### Speed Up APT

Add to playbook:

```yaml
- name: Update apt cache
  apt:
    update_cache: yes
    cache_valid_time: 3600  # Cache valid for 1 hour
```

## Integration with CI/CD

### GitLab CI Example

```yaml
deploy:
  stage: deploy
  script:
    - ansible-playbook -i inventory.ini playbook.yml
  only:
    - main
```

### GitHub Actions Example

```yaml
name: Deploy Workstation
on:
  push:
    branches: [ main ]
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Run Ansible
        run: ansible-playbook -i inventory.ini playbook.yml
```

## Related Projects

- [ansible-server-bootstrap](https://github.com/fabien-devops/ansible-server-bootstrap) - VPS security hardening
- [bash-devops-scripts](https://github.com/fabien-devops/bash-devops-scripts) - Bash automation utilities
- [Terraform-EKS](https://github.com/fabien-devops/Terraform-eks) - Kubernetes infrastructure

## Contributing

Contributions are welcome! Ideas for improvements:

- [ ] Add role-based organization
- [ ] Support for CentOS/RHEL
- [ ] Docker installation option
- [ ] Database server setup
- [ ] Automated backups configuration
- [ ] SSL certificate management
- [ ] Fail2ban integration
- [ ] Monitoring stack setup

## Common Use Cases

### Development Workstation
```bash
# Setup local development environment
ansible-playbook -i inventory.ini playbook.yml --limit localhost
```

### Web Server Fleet
```bash
# Deploy to all web servers
ansible-playbook -i inventory.ini playbook.yml
```

### Staging Environment
```bash
# Setup staging server
ansible-playbook -i inventory.ini playbook.yml --limit staging
```

## Changelog

### Version 1.0
- Initial release
- User creation and management
- System updates and package installation
- Custom MOTD setup
- Hostname and timezone configuration

## License

This project is provided as-is for educational and personal use.

## Author

**Fabien Andrianambinintsoa**

DevOps Engineer passionate about automation and infrastructure as code.

- **GitHub**: [fabien-devops](https://github.com/fabien-devops)
- **LinkedIn**: [Fabien Andrianambinintsoa](https://www.linkedin.com/in/fabien-andrianambinintsoa)
- **Email**: fabien.andrianambinintsoa@gmail.com

## Acknowledgments

- Ansible community for excellent documentation
- Ubuntu/Debian for stable platforms
- DevOps community for inspiration and best practices

---

⭐ *If you find this project useful, please consider giving it a star!*

*Built with ❤️ for efficient server automation*

