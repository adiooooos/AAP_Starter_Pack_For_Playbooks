# RHEL9 System Standardization Configuration

## Overview

This Ansible playbook automates the standardization of RHEL9 systems, providing a comprehensive solution for system configuration management. The playbook includes 7 core functional modules that ensure consistent, automated, and repeatable server configurations.

**Key Feature**: All configuration parameters are centralized in `group_vars/all.yml`, making it easy to customize the playbook for your specific environment without modifying the playbook code itself.

## Features

### ✅ User Management Automation

- Automatic admin user creation with customizable username and password
- Sudo privileges configuration (passwordless)
- Session persistence enablement
- Secure password settings

### ✅ System Configuration Standardization

- Hostname validation
- SELinux configuration (configurable mode: permissive/enforcing/disabled)
- System information collection

### ✅ Intelligent Firewall Configuration

- Batch port opening (fully configurable port list)
- TCP/UDP dual protocol support
- Configuration persistence
- Service reload with error handling

### ✅ Hosts File Standardization

- Fetch standard hosts from template server (configurable server and path)
- Automatic backup of original file
- Intelligent configuration merging
- Local entry protection

### ✅ Precise Time Synchronization Configuration

- Chrony service installation and configuration
- Configurable NTP servers (supports multiple servers)
- Network subnet allowlist configuration
- Configuration verification

### ✅ System Performance Optimization

- File descriptor optimization (configurable limits)
- Network parameter tuning (configurable sysctl parameters)
- Memory mapping optimization
- System limits configuration

### ✅ Comprehensive Verification Mechanism

- System information verification
- User configuration checking
- Network connectivity testing
- Service status confirmation
- Detailed error reporting and summary

## Supported Operating Systems

- **RHEL 9 only**

The playbook includes OS compatibility checks and will fail gracefully if run on unsupported systems.

## Project Structure

```
Ansible救火热线系列之(17)RHEL9标准化配置/
├── inventory                    # Host inventory configuration
├── group_vars/
│   └── all.yml                 # ⭐ Global variables configuration (CUSTOMIZE HERE)
├── rhel9_standardization_site.yml  # Main playbook file
├── ansible.cfg                 # Ansible configuration file
├── roles/                      # 7 functional roles
│   ├── user_management/        # User management
│   ├── system_config/          # System configuration
│   ├── firewall_config/        # Firewall configuration
│   ├── hosts_config/           # Hosts file standardization
│   ├── chrony_config/          # Time synchronization configuration
│   ├── system_optimization/    # System optimization
│   └── verification/           # Configuration verification
└── README.md                   # This file
```

## Prerequisites

1. **Ansible Installation**

   ```bash
   ansible --version
   ```

   Ensure Ansible 2.9+ is installed.
2. **SSH Key Authentication**

   ```bash
   ssh-keygen -t rsa -N "" -f ~/.ssh/id_rsa
   ssh-copy-id root@target_server_ip
   ```
3. **Required Collections**

   - `ansible.posix` (for firewalld and selinux modules)

   ```bash
   ansible-galaxy collection install ansible.posix
   ```

## Configuration Guide

### ⭐ Customization via `group_vars/all.yml`

**The most important file for customization is `group_vars/all.yml`.** This file contains all configurable parameters that control the behavior of the playbook. You can customize the entire playbook behavior by editing this single file.

### 1. Inventory Configuration

Edit the `inventory` file to add your target servers:

```ini
[rhel9]
ansible25.example.com
your-server.example.com

[template_servers]
template_server ansible_host=10.66.208.237
```

### 2. Customization via `group_vars/all.yml`

#### 2.1 Administrator User Configuration

**Purpose**: Configure the admin user that will be created on all target systems.

```yaml
# Administrator user configuration
admin_user: admin                    # Username for the admin account
admin_password: redhat               # Password for the admin account (will be hashed)
```

**Customization Examples**:

```yaml
# Example 1: Use a different username
admin_user: opsadmin
admin_password: SecureP@ssw0rd123

# Example 2: Use environment-specific credentials
admin_user: "{{ env_admin_user | default('admin') }}"
admin_password: "{{ env_admin_password | default('changeme') }}"
```

**Impact**:

- Creates a new user account with the specified username
- Sets the password (automatically hashed using SHA512)
- Configures passwordless sudo access
- Enables systemd user session persistence

---

#### 2.2 Firewall Ports Configuration

**Purpose**: Define which ports should be opened in the firewall (both TCP and UDP).

```yaml
# Firewall ports configuration
firewall_ports:
  - 50051
  - 53
  - 80
  - 443
  - 5432
  - 6379
  - 27199
  - 9000
  - 9001
  - 9002
  - 9003
```

**Customization Examples**:

```yaml
# Example 1: Add application-specific ports
firewall_ports:
  - 22      # SSH
  - 80      # HTTP
  - 443     # HTTPS
  - 3306    # MySQL
  - 8080    # Application port
  - 9090    # Monitoring port

# Example 2: Minimal production setup
firewall_ports:
  - 22      # SSH
  - 80      # HTTP
  - 443     # HTTPS

# Example 3: Development environment with many services
firewall_ports:
  - 22
  - 80
  - 443
  - 3306    # MySQL
  - 5432    # PostgreSQL
  - 6379    # Redis
  - 8080
  - 9000
  - 9200    # Elasticsearch
```

**Impact**:

- Opens all listed ports for both TCP and UDP protocols
- Configures permanent firewall rules (survives reboot)
- If firewalld is not running, ports are recorded as failed but playbook continues
- All failed ports are reported in the summary

**Note**: The playbook is robust - if firewalld service is not available, it will continue with other configurations and report firewall failures in the summary.

---

#### 2.3 NTP Servers Configuration

**Purpose**: Configure time synchronization servers for Chrony.

```yaml
# NTP servers configuration
ntp_servers:
  - ntp.aliyun.com
  - ntp1.aliyun.com
  - ntp2.aliyun.com
```

**Customization Examples**:

```yaml
# Example 1: Use internal NTP servers
ntp_servers:
  - ntp.internal.company.com
  - ntp-backup.internal.company.com

# Example 2: Use public NTP pools
ntp_servers:
  - 0.pool.ntp.org
  - 1.pool.ntp.org
  - 2.pool.ntp.org
  - 3.pool.ntp.org

# Example 3: Mixed internal and external
ntp_servers:
  - ntp.internal.company.com
  - time.google.com
  - pool.ntp.org
```

**Impact**:

- Configures Chrony to use these NTP servers
- Multiple servers provide redundancy
- Servers are configured with `iburst` option for faster initial sync

---

#### 2.4 Network Configuration

**Purpose**: Define which network subnets are allowed to synchronize time with this server.

```yaml
# Network configuration
network:
  allowed_subnets:
    - 10.66.0.0/16
    - 192.168.0.0/16
```

**Customization Examples**:

```yaml
# Example 1: Single internal network
network:
  allowed_subnets:
    - 10.0.0.0/8

# Example 2: Multiple internal networks
network:
  allowed_subnets:
    - 10.0.0.0/8
    - 172.16.0.0/12
    - 192.168.0.0/16

# Example 3: Specific subnets only
network:
  allowed_subnets:
    - 10.10.0.0/24
    - 10.20.0.0/24
```

**Impact**:

- Allows time synchronization requests from specified subnets
- Restricts NTP access for security
- Applied in Chrony configuration

---

#### 2.5 Template Server Configuration

**Purpose**: Configure the template server from which to fetch the standard `/etc/hosts` file.

```yaml
# Template server configuration
template_server:
  host: 10.66.208.237
  hosts_file: /etc/hosts
```

**Customization Examples**:

```yaml
# Example 1: Use a different template server
template_server:
  host: 192.168.1.100
  hosts_file: /etc/hosts

# Example 2: Use a custom hosts file location
template_server:
  host: template-server.example.com
  hosts_file: /opt/standard-configs/hosts.standard

# Example 3: Use inventory hostname
template_server:
  host: "{{ groups['template_servers'][0] }}"
  hosts_file: /etc/hosts
```

**Impact**:

- Fetches the hosts file from the specified server
- Merges with local host entries
- If template server is unreachable, playbook continues and reports the failure
- Automatically backs up the original hosts file

**Note**: The playbook handles template server failures gracefully - if the server is unreachable, it will skip hosts file configuration and continue with other tasks.

---

#### 2.6 System Optimization Parameters

**Purpose**: Configure system resource limits and kernel parameters for performance optimization.

##### System Limits

```yaml
# System optimization parameters
system_limits:
  - "prometheus soft nofile 65536"
  - "prometheus hard nofile 65536"
  - "* soft nofile 65536"
  - "* hard nofile 65536"
```

**Customization Examples**:

```yaml
# Example 1: Higher limits for high-performance systems
system_limits:
  - "* soft nofile 131072"
  - "* hard nofile 131072"
  - "* soft nproc 32768"
  - "* hard nproc 32768"

# Example 2: Application-specific limits
system_limits:
  - "nginx soft nofile 65536"
  - "nginx hard nofile 65536"
  - "postgres soft nofile 32768"
  - "postgres hard nofile 32768"
  - "* soft nofile 65536"
  - "* hard nofile 65536"

# Example 3: Minimal limits
system_limits:
  - "* soft nofile 32768"
  - "* hard nofile 32768"
```

##### Sysctl Parameters

```yaml
sysctl_params:
  net.core.somaxconn: 65535
  net.ipv4.tcp_max_syn_backlog: 65535
  vm.max_map_count: 262144
```

**Customization Examples**:

```yaml
# Example 1: High-performance web server
sysctl_params:
  net.core.somaxconn: 131070
  net.ipv4.tcp_max_syn_backlog: 131070
  net.ipv4.ip_local_port_range: "1024 65535"
  net.core.netdev_max_backlog: 5000
  vm.max_map_count: 262144

# Example 2: Database server optimization
sysctl_params:
  net.core.somaxconn: 65535
  net.ipv4.tcp_max_syn_backlog: 65535
  vm.max_map_count: 524288
  vm.swappiness: 1
  vm.dirty_ratio: 15
  vm.dirty_background_ratio: 5

# Example 3: Minimal optimization
sysctl_params:
  net.core.somaxconn: 4096
  net.ipv4.tcp_max_syn_backlog: 4096
```

**Impact**:

- System limits are written to `/etc/security/limits.conf`
- Sysctl parameters are written to `/etc/sysctl.d/99-ansible.conf`
- Changes take effect immediately (sysctl reload)
- System limits require new login sessions to take effect
- All changes are automatically backed up

---

#### 2.7 SELinux Configuration

**Purpose**: Configure SELinux mode and policy.

```yaml
# SELinux configuration
selinux:
  state: permissive
  policy: targeted
```

**Customization Examples**:

```yaml
# Example 1: Enforcing mode (more secure)
selinux:
  state: enforcing
  policy: targeted

# Example 2: Disabled (not recommended for production)
selinux:
  state: disabled
  policy: targeted

# Example 3: Permissive (recommended for troubleshooting)
selinux:
  state: permissive
  policy: targeted
```

**Impact**:

- Sets SELinux mode temporarily (immediate effect)
- Configures SELinux mode permanently in `/etc/selinux/config`
- **Note**: Changing SELinux state may require a system reboot to fully take effect
- The playbook will warn if a reboot is required

**Security Note**:

- `enforcing`: Blocks unauthorized access (recommended for production)
- `permissive`: Logs violations but doesn't block (good for troubleshooting)
- `disabled`: Completely disables SELinux (not recommended)

---

#### 2.8 Backup Configuration

**Purpose**: Control automatic backup behavior for configuration files.

```yaml
# Backup configuration
backup:
  enabled: true
  suffix: ".backup.$(date +%Y%m%d_%H%M%S)"
```

**Customization Examples**:

```yaml
# Example 1: Disable backups (not recommended)
backup:
  enabled: false

# Example 2: Custom backup suffix
backup:
  enabled: true
  suffix: ".backup.$(date +%Y%m%d)"

# Example 3: Always create backups
backup:
  enabled: true
  suffix: ".backup.$(date +%Y%m%d_%H%M%S)"
```

**Impact**:

- When enabled, creates backups before modifying configuration files
- Backup files are created with timestamp suffix
- Backups are created for:
  - `/etc/hosts`
  - `/etc/chrony.conf`
  - `/etc/selinux/config`
  - `/etc/security/limits.conf`
  - `/etc/sysctl.d/99-ansible.conf`

---

## Usage

### Full Execution

Execute the complete standardization playbook:

```bash
ansible-playbook rhel9_standardization_site.yml -i inventory
```

### Check Mode (Dry Run)

Test the playbook without making changes:

```bash
ansible-playbook rhel9_standardization_site.yml -i inventory -C
```

### Targeted Execution

Execute specific roles using tags:

```bash
# User management only
ansible-playbook rhel9_standardization_site.yml -i inventory --tags user_management

# Firewall configuration only
ansible-playbook rhel9_standardization_site.yml -i inventory --tags firewall_config

# Time synchronization only
ansible-playbook rhel9_standardization_site.yml -i inventory --tags chrony_config

# System optimization only
ansible-playbook rhel9_standardization_site.yml -i inventory --tags system_optimization
```

### Parallel Execution

For bulk deployment across multiple hosts:

```bash
ansible-playbook rhel9_standardization_site.yml -i inventory --forks 10
```

### Verbose Output

For detailed execution output:

```bash
ansible-playbook rhel9_standardization_site.yml -i inventory -v
# or
ansible-playbook rhel9_standardization_site.yml -i inventory -vvv
```

## Configuration Workflow

### Step-by-Step Customization Process

1. **Review Default Configuration**

   ```bash
   cat group_vars/all.yml
   ```
2. **Customize for Your Environment**

   - Edit `group_vars/all.yml` with your specific requirements
   - Test in check mode first: `ansible-playbook rhel9_standardization_site.yml -i inventory -C`
3. **Validate Configuration**

   - Review the playbook output for any warnings or errors
   - Check the configuration summary at the end of execution
4. **Execute on Target Systems**

   ```bash
   ansible-playbook rhel9_standardization_site.yml -i inventory
   ```
5. **Review Results**

   - Each role provides a detailed summary of its configuration status
   - Final summary provides an overview of all configurations
   - Address any reported errors or warnings

## Verification

After execution, verify the configuration:

```bash
# Check user configuration
id admin
sudo -l -U admin

# Check firewall configuration
firewall-cmd --list-all
firewall-cmd --list-ports

# Check time synchronization
chronyc sources
chronyc tracking

# Check system limits
ulimit -n
cat /etc/security/limits.conf | grep -v "^#"

# Check sysctl parameters
sysctl net.core.somaxconn
sysctl net.ipv4.tcp_max_syn_backlog
sysctl vm.max_map_count

# Check SELinux status
sestatus
getenforce
```

## Role Details

### user_management

Creates admin user with sudo privileges and enables session persistence. Reports success/failure status for each operation.

### system_config

Validates hostname configuration and configures SELinux. Handles SELinux configuration failures gracefully.

### firewall_config

Configures firewalld service and opens specified TCP/UDP ports. Continues execution even if firewalld is not available, reporting all failed ports in the summary.

### hosts_config

Fetches standard hosts file from template server and merges with local configuration. Handles template server connectivity issues gracefully.

### chrony_config

Installs and configures Chrony for time synchronization with NTP servers. Reports installation, configuration, and service status.

### system_optimization

Configures system limits and sysctl parameters for performance optimization. Reports configuration status for limits and sysctl parameters.

### verification

Comprehensive verification of all configuration changes. Provides detailed status for each verification check.

## Error Handling and Robustness

The playbook is designed with robust error handling:

- **Non-blocking Errors**: Most configuration failures are logged but don't stop execution
- **Detailed Reporting**: Each role provides a summary of successful and failed configurations
- **Graceful Degradation**: If a service is unavailable (e.g., firewalld), the playbook continues with other configurations
- **Final Summary**: Comprehensive summary at the end shows all errors and warnings

### Example Error Handling

If firewalld is not running:

- Port configuration tasks fail but are ignored
- Failed ports are collected and reported
- Playbook continues with other configurations
- Summary clearly shows which ports failed

If template server is unreachable:

- Hosts file fetch fails gracefully
- Playbook continues with other configurations
- Summary reports the hosts configuration failure

## Troubleshooting

### Common Issues

1. **OS Compatibility Error**

   - Ensure target systems are RHEL 9 only
   - Check `ansible_distribution` and `ansible_distribution_major_version` facts
2. **Firewall Configuration Failure**

   - Ensure `ansible.posix` collection is installed: `ansible-galaxy collection install ansible.posix`
   - Check firewalld service status: `systemctl status firewalld`
   - If firewalld is not running, start it: `systemctl start firewalld`
   - Review the firewall summary in the playbook output for failed ports
3. **Template Server Connection Failure**

   - Verify template server is accessible from control node
   - Check SSH connectivity: `ssh root@template_server_host`
   - Verify the hosts file path exists on template server
   - Review the hosts configuration summary in the playbook output
4. **SELinux Configuration Issues**

   - Some SELinux operations may require system reboot
   - Check SELinux status: `sestatus`
   - Review warnings in the playbook output about reboot requirements
5. **Package Installation Failures**

   - Ensure target systems have access to RHEL repositories
   - Check network connectivity to repositories
   - Verify subscription is active: `subscription-manager status`

### Debug Mode

Enable debug output for troubleshooting:

```bash
ansible-playbook rhel9_standardization_site.yml -i inventory -vvv
```

### Reviewing Configuration Summaries

Each role provides a detailed summary. Look for:

- ✅ Success indicators
- ❌ Failure indicators
- ⚠️ Warning messages
- Lists of failed configurations

## Best Practices

1. **Customize via `group_vars/all.yml`**: Never modify the playbook code directly. All customization should be done in `group_vars/all.yml`.
2. **Test in Check Mode First**: Always run with `-C` flag first to see what changes would be made.
3. **Version Control**: Keep your customized `group_vars/all.yml` in version control for different environments (dev, staging, production).
4. **Backup Configuration**: Keep `backup.enabled: true` in production environments.
5. **Incremental Deployment**: Use tags to deploy specific configurations incrementally.
6. **Review Summaries**: Always review the configuration summaries at the end of execution.
7. **Document Customizations**: Document any customizations made to `group_vars/all.yml` for your environment.

## Environment-Specific Configuration

### Development Environment Example

```yaml
admin_user: devadmin
admin_password: dev123
firewall_ports:
  - 22
  - 80
  - 443
  - 8080
  - 9000
selinux:
  state: permissive
backup:
  enabled: true
```

### Production Environment Example

```yaml
admin_user: opsadmin
admin_password: "{{ vault_admin_password }}"  # Use Ansible Vault
firewall_ports:
  - 22
  - 80
  - 443
  - 5432
selinux:
  state: enforcing
backup:
  enabled: true
sysctl_params:
  net.core.somaxconn: 131070
  net.ipv4.tcp_max_syn_backlog: 131070
  vm.max_map_count: 524288
```

### Using Ansible Vault for Sensitive Data

For production environments, use Ansible Vault to encrypt sensitive values:

```bash
# Create encrypted variable file
ansible-vault create group_vars/all.yml

# Edit encrypted file
ansible-vault edit group_vars/all.yml

# Run playbook with vault
ansible-playbook rhel9_standardization_site.yml -i inventory --ask-vault-pass
```

## Author

GCG AAP SSA Team + v3.01 Date 20260205

## License

📜 License Type: End User License Agreement (EULA)
🔒 Authorization: Through Subscription

## Version History

- **v3.01 (20260205)**: Initial English version with full role implementation
  - OS compatibility assertion for RHEL 9 only
  - Complete error handling with block-rescue
  - Full FQDN module names
  - Comprehensive verification mechanism
  - Robust error handling - continues execution on failures
  - Detailed configuration summaries for each role
