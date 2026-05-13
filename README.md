# Server Configurations Ansible Playbooks

## 01.server-first-config.yaml
This playbook (`server_setup.yml`) is an **interactive server configuration and hardening tool** that automates the setup of a Linux server. It handles user management, package installation, firewall configuration, and common services.

### Core Functionality

#### 1. **Interactive User Input** (vars_prompt)
Prompts the operator for:
- Username to create
- User password (hidden input with confirmation)
- Whether to grant sudo privileges
- Whether to configure custom package mirrors

#### 2. **User Management**
- Creates a new user account with home directory
- Sets password (encrypted with SHA512)
- Optionally adds user to `sudo` group
- Creates passwordless sudoers file if requested
- Creates a matching user group

#### 3. **Custom Mirror Configuration** (Optional)
If enabled, allows replacing default package repositories:
- **Ubuntu/Debian**: Backs up `/etc/apt/sources.list` and updates with custom mirror URLs
- **CentOS/RHEL**: Backs up `.repo` files and replaces mirrorlist with custom baseurls

#### 4. **System Updates & Upgrades**
- Updates package cache (apt/yum)
- Upgrades all packages to latest versions

#### 5. **Common Package Installation**
Installs a standard toolkit including:
- `curl, wget, vim, htop, git, net-tools`
- `docker.io, nginx`
- `python3, python3-pip`
- `ufw, fail2ban`

*Note: Docker is installed via official repositories, not just the distribution package*

#### 6. **Service Management**
- Starts and enables **Docker** daemon
- Adds the new user to `docker` group
- Starts and enables **Nginx** web server

#### 7. **Firewall Configuration** (Ubuntu/Debian only)
- Configures UFW to allow ports: **22 (SSH), 80 (HTTP), 443 (HTTPS)**
- Enables UFW with default deny policy

#### 8. **Verification Output**
Displays confirmation messages showing:
- User creation status and sudo privileges
- Docker version
- Nginx version

##### Supported Operating Systems
- **Ubuntu / Debian** (apt-based)
- **CentOS / RHEL / Rocky / AlmaLinux** (yum-based)

##### Key Features
- **Idempotent**: Safe to run multiple times
- **Distro-aware**: Uses correct package manager for each OS
- **Backup creation**: Original config files are backed up before modification
- **Passwordless sudo**: Option available for automation convenience
- **Security-first**: Enables firewall and installs fail2ban