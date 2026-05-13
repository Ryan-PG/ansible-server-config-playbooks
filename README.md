# Linux Server Configurations Ansible Playbooks
Into this repo, I try to put some ansible playbooks for handling and configuring linux servers.

### Prerequisites

#### 1. **Configure Inventory/Hosts File**
Before running any playbook, you MUST specify the target servers in your Ansible inventory:

**Option A: Edit the playbook directly**
```yaml
# In server_setup.yml, change the 'hosts' parameter:
- name: Server Setup and Configuration
  hosts: your-server-hostname-or-group  # Change this from 'all' to your specific host(s)
```

**Option B: Use Ansible inventory file (recommended)**
Create or edit `/etc/ansible/hosts` or a custom inventory file:
```ini
[web_servers]
192.168.1.10
192.168.1.11

[db_servers]
192.168.1.20

[all:vars]
ansible_user=root
ansible_ssh_private_key_file=~/.ssh/id_rsa
```

Then run with specific inventory:
```bash
ansible-playbook -i inventory.ini server_setup.yml
```

**Option C: Specify hosts at runtime**
```bash
# Run against a single host
ansible-playbook server_setup.yml --limit 192.168.1.10

# Run against a group from inventory
ansible-playbook server_setup.yml --limit web_servers

# Override hosts parameter
ansible-playbook server_setup.yml -e 'hosts=production-server'
```

> **⚠️ IMPORTANT**: The playbooks currently targets `hosts: all`. You must change this to your specific server hostname(s) or inventory group, otherwise it will run on ALL hosts in your Ansible configuration!



## 01.server-first-config.yaml
This playbook (`server_setup.yml`) is an **interactive server configuration and hardening tool** that automates the setup of a Linux server. It handles user management, package installation, firewall configuration, and common services.

#### **Interactive User Input** (vars_prompt)
Prompts the operator for:
- Username to create
- User password (hidden input with confirmation)
- Whether to grant sudo privileges
- Whether to configure custom package mirrors
- Whether to configure custom Docker mirror/registry

#### **User Management**
- Creates a new user account with home directory
- Sets password (encrypted with SHA512)
- Optionally adds user to `sudo` group
- Creates passwordless sudoers file if requested
- Creates a matching user group

#### **Custom Package Mirror Configuration** (Optional)
If enabled, allows replacing default package repositories:
- **Ubuntu/Debian**: Backs up `/etc/apt/sources.list` and updates with custom mirror URLs
- **CentOS/RHEL**: Backs up `.repo` files and replaces mirrorlist with custom baseurls

#### **Custom Docker Mirror Configuration** (Optional)
If enabled, allows configuring Docker registry mirrors for faster pulls:
- Prompts for primary Docker registry mirror URL (e.g., `https://mirror.gcr.io`, `https://docker-mirror.example.com`)
- Optionally accepts additional mirrors as comma-separated list
- Creates `/etc/docker/daemon.json` with registry-mirrors configuration
- Configures Docker daemon optimizations:
  - JSON file logging with rotation (max 10MB, keep 3 files)
  - Overlay2 storage driver for better performance
- Restarts Docker service automatically after configuration
- Verifies and displays mirror configuration status

#### **System Updates & Upgrades**
- Updates package cache (apt/yum)
- Upgrades all packages to latest versions

#### **Common Package Installation**
Installs a standard toolkit including:
- `curl, wget, vim, htop, git, net-tools`
- `docker.io, nginx`
- `python3, python3-pip`
- `ufw, fail2ban`

*Note: Docker is installed via official repositories, not just the distribution package*

#### **Service Management**
- Starts and enables **Docker** daemon
- Adds the new user to `docker` group
- Starts and enables **Nginx** web server

#### **Firewall Configuration** (Ubuntu/Debian only)
- Configures UFW to allow ports: **22 (SSH), 80 (HTTP), 443 (HTTPS)**
- Enables UFW with default deny policy

#### **Verification Output**
Displays confirmation messages showing:
- User creation status and sudo privileges
- Docker version
- Docker mirror configuration (if enabled)
- Nginx version

### Running the Playbook

#### Step 1: Configure your target hosts
```bash
# Either edit the playbook
vim server_setup.yml
# Change 'hosts: all' to 'hosts: your-server-name'

# Or prepare an inventory file
echo "192.168.1.100 ansible_user=root" > inventory.ini
```

#### Step 2: Test connectivity
```bash
ansible -i inventory.ini all -m ping
```

#### Step 3: Run the playbook
```bash
# Using inventory file
ansible-playbook -i inventory.ini server_setup.yml

# Directly against a host
ansible-playbook server_setup.yml --limit 192.168.1.100

# With verbose output for debugging
ansible-playbook -i inventory.ini server_setup.yml -v
```

##### Supported Operating Systems
- **Ubuntu / Debian** (apt-based)
- **CentOS / RHEL / Rocky / AlmaLinux** (yum-based)

##### Key Features
- **Idempotent**: Safe to run multiple times
- **Distro-aware**: Uses correct package manager for each OS
- **Backup creation**: Original config files are backed up before modification
- **Passwordless sudo**: Option available for automation convenience
- **Security-first**: Enables firewall and installs fail2ban
- **Docker optimization**: Registry mirrors for faster pulls and better performance
- **Handler-based**: Docker service restarts only when configuration changes

##### Example Docker Mirror Usage

**Primary mirror examples:**
- Google Container Registry: `https://mirror.gcr.io`
- Docker Hub mirror: `https://docker-mirror.example.com`
- Aliyun mirror (China): `https://registry.cn-hangzhou.aliyuncs.com`

**Additional mirrors example:**
- `https://mirror.azure.cn/docker-engine,https://hub-mirror.c.163.com`

The configured mirrors will be used by Docker daemon for all image pulls, significantly improving download speeds in regions with slow access to Docker Hub.

##### Common Use Cases

1. **Air-gapped environments**: Configure internal Docker registry mirror
2. **Geographic optimization**: Use region-specific mirrors for faster pulls
3. **Corporate networks**: Route through corporate Docker proxy/registry
4. **Rate limit avoidance**: Bypass Docker Hub pull rate limits using mirrors

##### Troubleshooting

**Issue**: "No hosts matched" error
- **Solution**: Verify your hosts parameter in the playbook or inventory file matches the target server names/IPs

**Issue**: Permission denied (SSH)
- **Solution**: Ensure you have SSH access and specify the correct `ansible_user` in inventory

**Issue**: Docker service fails to restart
- **Solution**: Check `/etc/docker/daemon.json` syntax and verify mirror URLs are accessible