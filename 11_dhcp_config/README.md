# Linux DHCP Automation

## Overview

This Ansible Role-based solution provides automated DHCP server deployment and client configuration across multiple Linux hosts. It enables deployment of:

- **DHCP Server** supporting both **DHCPv4** and **DHCPv6**
- **DHCP Client** configuration using Red Hat system roles

The implementation follows AAP Starter Pack guidelines with FQDN modules, OS compatibility assertions, robust error handling, and clear debugging support.

## Features

- **Role-Based Architecture**: Modular, reusable, and maintainable design
- **OS Compatibility**:
  - DHCP Server: RHEL/CentOS 8/9 (DHCP server package not available in RHEL 7)
  - DHCP Client: RHEL/CentOS 7/8/9
- **Dual Protocol Support**: Both DHCPv4 and DHCPv6 configuration
- **File-Based Configuration**: Static configuration files for simplicity and clarity
- **Comprehensive Validation**: OS compatibility checks and parameter validation
- **Error Handling**: Robust error handling with `block-rescue` for critical changes
- **Firewall Management**: Automatic firewalld configuration for DHCP services
- **SELinux Support**: Proper SELinux context settings for configuration files
- **Standardized Client Configuration**: Uses `rhel-system-roles.network` for client setup

## Files

- `deploy_dhcp_server_site.yml` - Main playbook for DHCP server deployment
- `configure_dhcp_client_site.yml` - Main playbook for DHCP client configuration
- `roles/dhcp_server/` - DHCP server role
  - `defaults/main.yml` - Default variables
  - `tasks/main.yml` - Core role tasks
  - `handlers/main.yml` - Service restart handlers
  - `files/dhcpd.conf` - DHCPv4 configuration file
  - `files/dhcpd6.conf` - DHCPv6 configuration file
- `README.md` - This file

## Prerequisites

- Ansible 2.9 or later
- Ansible control node with network access to target hosts
- Target hosts:
  - **DHCP Server**: RHEL 8/9, CentOS 8/9, Rocky Linux 8/9, AlmaLinux 8/9, or Oracle Linux 8/9
  - **DHCP Client**: RHEL 7/8/9, CentOS 7/8/9, Rocky Linux 8/9, AlmaLinux 8/9, or Oracle Linux 7/8/9
- SSH access to target hosts with root privileges (or sudo access)
- **For DHCP client**: `rhel-system-roles.network` collection must be installed on control node (see Installation section below)
- firewalld service (optional, can be disabled via variables)

## Installation

### Install rhel-system-roles.network Collection

Before running the DHCP client playbook, you must install the `rhel-system-roles.network` collection on your Ansible control node.

#### Method 1: Using ansible-galaxy (Recommended)

```bash
ansible-galaxy collection install rhel-system-roles.network
```

#### Method 2: Using yum/dnf (RHEL/CentOS)

```bash
# RHEL 8/9
dnf install rhel-system-roles

# CentOS 8/9
dnf install linux-system-roles
```

#### Method 3: Install from requirements.yml

Create a `requirements.yml` file:

```yaml
---
collections:
  - name: rhel-system-roles.network
    version: ">=1.0.0"
```

Then install:

```bash
ansible-galaxy collection install -r requirements.yml
```

#### Verify Installation

After installation, verify the collection is available:

```bash
ansible-galaxy collection list | grep rhel-system-roles
```

You should see output like:

```
rhel-system-roles.network    <version>
```

## Usage

### 1. Prepare Inventory File

Create an inventory file (e.g., `inventory`) with your target hosts organized by function:

```ini
[dhcp_servers]
servera.example.com

[dhcp_clients]
serverb.example.com
serverc.example.com
```

**Important Notes**:

- DHCP server requires RHEL/CentOS 8/9 (not available in RHEL 7)
- DHCP client works on RHEL/CentOS 7/8/9
- You can organize hosts into functional groups as needed
- The playbook will validate OS compatibility before execution

### 2. Customize Configuration (Optional)

#### DHCP Server Configuration

Edit the configuration files directly:

- **DHCPv4**: `roles/dhcp_server/files/dhcpd.conf`
- **DHCPv6**: `roles/dhcp_server/files/dhcpd6.conf`

Adjust IP address ranges, gateways, DNS servers, MAC/DUID bindings, and lease times to match your environment.

#### DHCP Client Configuration

Edit `configure_dhcp_client_site.yml` to customize client settings:

```yaml
vars:
  network_connections:
    - name: "dhcp-connection"
      interface_name: eth1  # Change to your interface name
      type: ethernet
      ip:
        dhcp4: yes
        auto6: yes
      state: up
```

### 3. Run the Playbooks

#### Deploy DHCP Server

```bash
ansible-playbook -i inventory deploy_dhcp_server_site.yml
```

This will:

- Install DHCP server package
- Deploy `dhcpd.conf` and `dhcpd6.conf` configuration files
- Start and enable dhcpd and dhcpd6 services
- Configure firewalld to allow DHCP and DHCPv6 services

#### Configure DHCP Client

```bash
ansible-playbook -i inventory configure_dhcp_client_site.yml
```

This will:

- Configure network interface to use DHCP (IPv4) and auto6 (IPv6)
- Use `rhel-system-roles.network` for standardized configuration

## Playbook Structure

### DHCP Server Playbook (`deploy_dhcp_server_site.yml`)

- **OS Compatibility Validation**: Checks for RHEL/CentOS 8/9
- **Package Installation**: Installs DHCP server package
- **Configuration Deployment**: Copies `dhcpd.conf` and `dhcpd6.conf` to `/etc/dhcp/`
- **Service Management**: Starts and enables dhcpd and dhcpd6 services
- **Firewall Configuration**: Configures firewalld to allow DHCP services

### DHCP Client Playbook (`configure_dhcp_client_site.yml`)

- **OS Compatibility Validation**: Checks for RHEL/CentOS 7/8/9
- **Network Configuration**: Uses `rhel-system-roles.network` to configure DHCP client
- **Standardized Approach**: Red Hat recommended method for network configuration

## Customization Examples

### Example 1: Customize DHCPv4 Subnet

Edit `roles/dhcp_server/files/dhcpd.conf`:

```conf
subnet 10.0.0.0 netmask 255.255.255.0 {
  range 10.0.0.100 10.0.0.200;
  option routers 10.0.0.1;
  option domain-name-servers 10.0.0.1, 8.8.8.8;
  default-lease-time 3600;
  max-lease-time 86400;
}
```

### Example 2: Customize DHCPv6 Subnet

Edit `roles/dhcp_server/files/dhcpd6.conf`:

```conf
subnet6 2001:db8::/64 {
  range6 2001:db8::100 2001:db8::200;
  option dhcp6.name-servers 2001:db8::1;
  default-lease-time 3600;
  max-lease-time 86400;
}
```

### Example 3: Disable Firewall Management

Edit `roles/dhcp_server/defaults/main.yml`:

```yaml
dhcp_manage_firewall: false
```

Or use extra vars:

```bash
ansible-playbook -i inventory deploy_dhcp_server_site.yml -e "dhcp_manage_firewall=false"
```

### Example 4: Customize Client Interface

Edit `configure_dhcp_client_site.yml`:

```yaml
vars:
  network_connections:
    - name: "dhcp-connection"
      interface_name: ens192  # Change to your interface
      type: ethernet
      ip:
        dhcp4: yes
        auto6: yes
      state: up
```

## Best Practices

- **Test First**: Use `--check` mode to preview changes:

  ```bash
  ansible-playbook -i inventory deploy_dhcp_server_site.yml --check
  ```
- **Validate Configuration**: After deployment, validate DHCP configuration:

  ```bash
  # Check DHCPv4 configuration
  dhcpd -t -cf /etc/dhcp/dhcpd.conf

  # Check DHCPv6 configuration
  dhcpd6 -t -cf /etc/dhcp/dhcpd6.conf
  ```
- **Test DHCP Functionality**: Verify DHCP server is working:

  ```bash
  # Check service status
  systemctl status dhcpd
  systemctl status dhcpd6

  # Check leases
  cat /var/lib/dhcpd/dhcpd.leases
  ```
- **Backup Configuration**: Consider backing up DHCP configuration before changes
- **Test in Non-Production**: Test DHCP configurations in non-production environments first
- **Document Changes**: Document DHCP configuration changes for audit and troubleshooting

## Troubleshooting

### Playbook fails with "Unsupported operating system"

- **For DHCP Server**: Verify target host OS is RHEL/CentOS 8/9 (DHCP server not available in RHEL 7)
- **For DHCP Client**: Verify target host OS is RHEL/CentOS 7/8/9
- Check `ansible_distribution` and `ansible_distribution_major_version` facts
- Remove any unsupported hosts from the inventory

### "firewalld service is not available"

- **Disable firewall management** by setting:

  - `dhcp_manage_firewall: false` (in `roles/dhcp_server/defaults/main.yml`)
- Or install and enable firewalld:

  ```bash
  yum install firewalld
  systemctl enable --now firewalld
  ```

### "dhcpd/dhcpd6 service fails to start"

- **Check service status:**

  ```bash
  systemctl status dhcpd
  systemctl status dhcpd6
  journalctl -u dhcpd
  journalctl -u dhcpd6
  ```
- **Validate configuration syntax:**

  ```bash
  # DHCPv4
  dhcpd -t -cf /etc/dhcp/dhcpd.conf

  # DHCPv6
  dhcpd6 -t -cf /etc/dhcp/dhcpd6.conf
  ```
- **Check SELinux contexts:**

  ```bash
  ls -Z /etc/dhcp/dhcpd.conf
  ls -Z /etc/dhcp/dhcpd6.conf
  ```

### "Clients cannot obtain IP addresses"

- **Check DHCP server is listening:**

  ```bash
  ss -tulpn | grep :67
  ss -tulpn | grep :547
  ```
- **Check firewall rules:**

  ```bash
  firewall-cmd --list-services
  firewall-cmd --list-ports
  ```
- **Check DHCP server logs:**

  ```bash
  journalctl -u dhcpd -f
  journalctl -u dhcpd6 -f
  ```
- **Verify IP address ranges are correct** in configuration files

### "rhel-system-roles.network not found" or "ERROR! the role 'rhel-system-roles.network' was not found"

This error occurs when the `rhel-system-roles.network` collection is not installed on the Ansible control node.

**Solution 1: Install using ansible-galaxy (Recommended)**

```bash
ansible-galaxy collection install rhel-system-roles.network
```

**Solution 2: Install using yum/dnf (RHEL/CentOS)**

```bash
# RHEL 8/9
dnf install rhel-system-roles

# CentOS 8/9
dnf install rhel-system-roles
```

**Solution 3: Install from requirements.yml**

Create a `requirements.yml` file in your project directory:

```yaml
---
collections:
  - name: rhel-system-roles.network
    version: ">=1.0.0"
```

Then install:

```bash
ansible-galaxy collection install -r requirements.yml
```

**Verify Installation**

After installation, verify the collection is available:

```bash
ansible-galaxy collection list | grep rhel-system-roles
```

Expected output:

```
rhel-system-roles.network    <version>
```

**Check Collection Path**

If the collection is still not found, check where Ansible is looking for collections:

```bash
ansible-config dump | grep COLLECTIONS_PATHS
```

Ensure the collection is installed in one of the paths listed.

**For AAP (Ansible Automation Platform) Users**

If you're using AAP, you may need to:

1. Install the collection in the AAP project directory, or
2. Install it globally on the AAP controller, or
3. Use a `requirements.yml` file in your project and let AAP sync it automatically

Check your AAP project settings for collection management options.

### "SELinux context issues"

- **Check current contexts:**

  ```bash
  ls -Z /etc/dhcp/dhcpd.conf
  ls -Z /etc/dhcp/dhcpd6.conf
  ```
- **Restore contexts if needed:**

  ```bash
  restorecon -R /etc/dhcp/
  ```
- **Customize SELinux type** in `roles/dhcp_server/defaults/main.yml`:

  ```yaml
  dhcp_config_setype: dhcp_etc_t
  ```

## Example Output

### Deploy DHCP Server

```
PLAY [Deploy and configure DHCPv4 and DHCPv6 servers] ********************

TASK [Validate OS compatibility] ****************************************
ok: [servera.example.com] => {
    "msg": "OS compatibility check passed: RedHat 8"
}

TASK [dhcp_server : Ensure DHCP server package is installed] ************
changed: [servera.example.com]

TASK [dhcp_server : Deploy DHCPv4 configuration file] ******************
changed: [servera.example.com]

TASK [dhcp_server : Deploy DHCPv6 configuration file] ******************
changed: [servera.example.com]

TASK [dhcp_server : Start and enable DHCP services] ********************
changed: [servera.example.com] => (item=dhcpd)
changed: [servera.example.com] => (item=dhcpd6)

TASK [dhcp_server : Ensure firewall allows DHCP services] ***************
changed: [servera.example.com] => (item=dhcp)
changed: [servera.example.com] => (item=dhcpv6)

RUNNING HANDLER [dhcp_server : restart dhcpd] ***************************
changed: [servera.example.com]

RUNNING HANDLER [dhcp_server : restart dhcpd6] *************************
changed: [servera.example.com]

PLAY RECAP ****************************************************************
servera.example.com      : ok=8    changed=7    unreachable=0    failed=0
```

### Configure DHCP Client

```
PLAY [Configure DHCP client] ********************************************

TASK [Validate OS compatibility] ****************************************
ok: [serverb.example.com] => {
    "msg": "OS compatibility check passed: RedHat 8"
}

TASK [rhel-system-roles.network : Configure network connections] ********
changed: [serverb.example.com]

PLAY RECAP ****************************************************************
serverb.example.com      : ok=2    changed=1    unreachable=0    failed=0
```

## Role Design Philosophy

This solution uses Ansible Role architecture with the following design principles:

1. **Modularity**: Separate role for DHCP server, standardized client configuration
2. **File-Based Configuration**: Static configuration files for simplicity and clarity
3. **Dual Protocol Support**: Both DHCPv4 and DHCPv6 in a single role
4. **Reusability**: Role can be used in any playbook or project
5. **Maintainability**: Easy to update and extend
6. **Best Practices**: FQDN modules, OS compatibility checks, error handling
7. **Standardized Client**: Uses Red Hat system roles for client configuration

## Security Considerations

- **Root Access Required**: This playbook requires root privileges to modify DHCP configurations
- **Network Service**: DHCP is a critical network service - test thoroughly before production
- **Firewall Rules**: Playbooks automatically configure firewalld - review rules for your environment
- **SELinux**: Proper SELinux contexts are set - ensure your policy allows these contexts
- **IP Address Management**: Review IP address ranges and reservations carefully
- **Test First**: Always test in non-production environments
- **Backup Configuration**: Consider backing up DHCP configuration before changes

## Author

GCG AAP SSA Team + v3.01 20260217

## License

📜 License Type: End User License Agreement (EULA)
🔒 Authorization: Subscription-based License
