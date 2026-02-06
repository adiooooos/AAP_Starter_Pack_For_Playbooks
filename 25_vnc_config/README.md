# Automated TigerVNC Server Configuration

## Overview

This playbook automates the installation and configuration of TigerVNC server on RHEL/CentOS systems. It provides a complete, enterprise-grade solution for deploying remote desktop access across your infrastructure, eliminating manual configuration steps and ensuring consistent, secure VNC deployments.
### screenshots
<img width="2031" height="1060" alt="image" src="https://github.com/user-attachments/assets/8f6c7aed-8e93-4fcd-a598-138d7d1d9059" />

## Features

- **Complete Automation**: End-to-end VNC server deployment from package installation to service startup
- **Enterprise Security**: Secure password handling with Ansible Vault support, firewall configuration
- **OS Compatibility**: Supports RHEL 7/8/9 and CentOS 7/8/9 with automatic validation
- **Flexible Configuration**: Variable-driven design supports multiple users, resolutions, and desktop environments
- **Robust Error Handling**: Comprehensive error handling with block-rescue structures ensures execution continues even if individual tasks fail
- **Idempotent Design**: Safe to run multiple times without side effects
- **Bulk Execution Optimized**: Efficient package installation for large-scale deployments
- **Comprehensive Validation**: Input parameter validation prevents configuration errors

## Prerequisites

- Ansible 2.9 or later
- Ansible control node with network access to target hosts
- Target hosts: RHEL 7/8/9 or CentOS 7/8/9
- SSH access to target hosts (password or key-based authentication)
- Sudo/root privileges on target hosts
- Internet access or configured YUM/DNF repository for TigerVNC packages
- Target hosts must have access to Red Hat software repositories (for TigerVNC and desktop environment installation)
- `ansible.posix` collection installed (for firewalld module)

## Installation

### Install Required Collections

```bash
# Install ansible.posix collection for firewalld module
ansible-galaxy collection install ansible.posix
```

## Files

- `vncconfig_site.yml` - Main VNC configuration playbook
- `inventory` - Host inventory file
- `ansible.cfg` - Ansible configuration file
- `README.md` - This file

## Usage

### 1. Prepare Inventory File

Edit the `inventory` file to define your target hosts. The playbook uses the `vnc_servers` group:

```ini
[vnc_servers]
# Define target hosts for VNC server configuration here
# Example:
# rhel-server-01 ansible_host=192.168.1.100
# rhel-server-02 ansible_host=192.168.1.101

# Using the standard inventory format:
[rhel7]
Test-RHEL-7.9-1
Test-RHEL-7.9-2
Test-RHEL-7.9-3

[rhel8]
TEST-RHEL-8.9-1

[rhel9]
ansible25.example.com

# Group all RHEL hosts as vnc_servers
[vnc_servers:children]
rhel7
rhel8
rhel9
```

### 2. Configure Variables

The playbook uses the following default variables (defined in the playbook):

```yaml
rhel_major_version: "8"          # RHEL major version for validation
vnc_user: "admin01"              # VNC username (must exist or will be created)
vnc_geometry: "1280x1024"        # Desktop resolution
vnc_display_number: "2"          # Display number (corresponds to port 5902)
vnc_desktop_session: "gnome"     # Desktop session type (gnome, xfce, etc.)
vnc_password: "YourSecretPassword"  # VNC password (use Vault for security!)
```

**Security Best Practice**: Use Ansible Vault to encrypt the VNC password:

```bash
# Encrypt the password
ansible-vault encrypt_string 'YourSecretPassword' --name 'vnc_password'

# Then use it in a vars file or command line
```

### 3. Run the Playbook

```bash
# Dry run first (recommended) - check what would be done
ansible-playbook vncconfig_site.yml -i inventory --check

# Execute VNC configuration with default variables
ansible-playbook vncconfig_site.yml -i inventory

# Execute with custom variables
ansible-playbook vncconfig_site.yml -i inventory \
  -e "vnc_user=myuser" \
  -e "vnc_geometry=1920x1080" \
  -e "vnc_display_number=3" \
  -e "vnc_desktop_session=xfce"

# Execute with Vault-encrypted password
ansible-playbook vncconfig_site.yml -i inventory --ask-vault-pass

# Run with increased parallelism for bulk execution
ansible-playbook vncconfig_site.yml -i inventory --forks 10

# Configure specific host or group
ansible-playbook vncconfig_site.yml -i inventory --limit rhel9
```

### 4. Connect to VNC Server

After successful execution, connect to the VNC server using a VNC client:

```bash
# Using display number
vncviewer <hostname>:2

# Using full port number
vncviewer <hostname>:5902

# Example
vncviewer Test-RHEL-8.9-1:2
```

## What Gets Configured

The playbook performs the following tasks:

1. **OS Compatibility Validation**: Verifies RHEL/CentOS 7/8/9 compatibility
2. **Input Parameter Validation**: Validates all configuration variables
3. **User Management**: Ensures VNC user exists (creates if missing)
4. **Package Installation**: Installs TigerVNC server and workstation desktop environment
5. **System Configuration**:
   - Configures `/etc/tigervnc/vncserver.users` for user mapping
   - Configures `/etc/tigervnc/vncserver-config-defaults` for default session
6. **User Configuration**:
   - Creates `~/.vnc` directory with proper permissions
   - Configures `~/.vnc/config` with resolution settings
7. **Security Configuration**:
   - Sets VNC password securely (using `vncpasswd`)
   - Configures firewall rules for VNC service
8. **Service Management**:
   - Enables and starts `vncserver@:X.service`
   - Verifies service status

## Configuration Variables

### Required Variables

- `vnc_user`: Username for VNC session (string, must exist or will be created)
- `vnc_display_number`: Display number (integer, 1-99, corresponds to port 5900+X)
- `vnc_password`: VNC connection password (string, minimum 6 characters, use Vault!)

### Optional Variables

- `rhel_major_version`: RHEL major version for validation (default: "8")
- `vnc_geometry`: Desktop resolution (default: "1280x1024", format: WIDTHxHEIGHT)
- `vnc_desktop_session`: Desktop session type (default: "gnome", options: gnome, xfce, etc.)

### Variable Examples

```yaml
# High-resolution configuration
vnc_geometry: "1920x1080"

# Multiple users on same host (run playbook multiple times with different display numbers)
vnc_user: "user1"
vnc_display_number: "2"

vnc_user: "user2"
vnc_display_number: "3"

# Lightweight desktop environment
vnc_desktop_session: "xfce"
```

## Advanced Usage

### Multi-User Configuration

Configure VNC for multiple users on the same host:

```bash
# Configure user1 on display :2
ansible-playbook vncconfig_site.yml -i inventory \
  -e "vnc_user=user1" \
  -e "vnc_display_number=2"

# Configure user2 on display :3
ansible-playbook vncconfig_site.yml -i inventory \
  -e "vnc_user=user2" \
  -e "vnc_display_number=3"
```

### Using Group Variables

Create `group_vars/vnc_servers.yml`:

```yaml
vnc_user: "admin01"
vnc_geometry: "1920x1080"
vnc_display_number: "2"
vnc_desktop_session: "gnome"
# Use Vault for password!
vnc_password: !vault |
  $ANSIBLE_VAULT;1.1;AES256
  ...
```

### Using Host Variables

Create `host_vars/<hostname>.yml` for host-specific configurations.

## Security Best Practices

1. **Use Ansible Vault**: Always encrypt VNC passwords using Ansible Vault
   ```bash
   ansible-vault encrypt_string 'YourPassword' --name 'vnc_password'
   ```

2. **Limit VNC Access**: Configure firewall rules to restrict VNC access to specific IP addresses
   ```bash
   firewall-cmd --add-rich-rule='rule family="ipv4" source address="192.168.1.0/24" service name="vnc-server" accept' --permanent
   ```

3. **Use SSH Tunneling**: For enhanced security, tunnel VNC through SSH
   ```bash
   ssh -L 5902:localhost:5902 user@vnc-server
   vncviewer localhost:2
   ```

4. **Regular Password Rotation**: Periodically update VNC passwords

5. **Monitor VNC Connections**: Regularly check VNC service logs and connections

## Troubleshooting

### VNC Service Not Starting

```bash
# Check service status
systemctl status vncserver@:2.service

# Check service logs
journalctl -u vncserver@:2.service -f

# Verify user mapping
cat /etc/tigervnc/vncserver.users

# Check user configuration
ls -la /home/<user>/.vnc/
```

### Connection Issues

1. **Firewall**: Verify firewall rules are configured
   ```bash
   firewall-cmd --list-services | grep vnc
   ```

2. **SELinux**: Check SELinux status (may need to configure SELinux policies)
   ```bash
   getenforce
   ```

3. **Network**: Verify network connectivity and port accessibility
   ```bash
   telnet <hostname> 5902
   ```

### Password Issues

If password authentication fails:

```bash
# Reset password manually
su - <vnc_user>
vncpasswd
```

### Desktop Session Issues

If desktop session doesn't start:

1. Verify desktop environment is installed
   ```bash
   rpm -qa | grep -E "gnome|xfce"
   ```

2. Check session configuration
   ```bash
   cat /etc/tigervnc/vncserver-config-defaults
   cat /home/<user>/.vnc/config
   ```

## Error Handling

The playbook includes comprehensive error handling:

- **OS Compatibility**: Fails fast if OS is not supported
- **Parameter Validation**: Validates all input parameters before execution
- **Task Resilience**: Individual task failures don't stop the entire playbook
- **Block-Rescue**: Critical operations use block-rescue for graceful error handling
- **Status Reporting**: Each major step includes debug output for troubleshooting

## Performance Considerations

- **Bulk Execution**: Package installation is optimized for multiple hosts
- **Idempotency**: Safe to run multiple times, only changes what's needed
- **Parallel Execution**: Use `--forks` option for faster execution on multiple hosts

## Limitations

- Requires desktop environment installation (workstation group or specific DE packages)
- VNC password must be at least 6 characters
- Display number must be between 1 and 99
- Desktop session type must be available on the system

## Support

For issues, questions, or contributions, please refer to the Ansible Automation Platform documentation or contact the GCG AAP SSA Team.

## License

📜 License Type: End User License Agreement (EULA)  
🔒 Authorization: Subscription-based License

---

**Author**: GCG AAP SSA Team + v3.01 20260217


