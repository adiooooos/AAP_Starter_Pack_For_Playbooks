# Automated NetworkManager Management

## Overview

This Ansible Role-based solution provides automated NetworkManager configuration management across multiple Linux hosts. It enables declarative network configuration management through a centralized control panel, supporting view, add, modify, control, and delete operations with precise tag-based execution control.

## Features

- **Role-Based Architecture**: Modular, reusable, and maintainable design
- **OS Compatibility**: Supports RHEL 7/8/9, CentOS 7/8/9, Rocky Linux 8/9, AlmaLinux 8/9, and Oracle Linux 7/8/9
- **Declarative Configuration**: Define desired network state in a single control panel file
- **Tag-Based Execution**: Precise control through tags (view, add, modify, control, delete)
- **Modular Design**: Separate task files for each operation type
- **Support for Static and DHCP**: Configure both static IP and DHCP connections
- **Network Connection Management**: Full lifecycle management (create, modify, activate, delete)

## Files

- `networkmanager_management_site.yml` - Main playbook file
- `roles/networkmanager_cli/` - NetworkManager CLI role
  - `defaults/main.yml` - Default variables
  - `vars/main.yml` - **Main Control Panel** (edit this file to configure network operations)
  - `tasks/main.yml` - Intelligent dispatch office (task scheduler)
  - `tasks/view.yml` - View network information tasks
  - `tasks/add.yml` - Add network connection tasks
  - `tasks/modify.yml` - Modify network connection tasks
  - `tasks/control.yml` - Control network connection tasks (activate/deactivate)
  - `tasks/delete.yml` - Delete network connection tasks
- `README.md` - This file

## Prerequisites

- Ansible 2.9 or later
- Ansible control node with network access to target hosts
- Target hosts: RHEL 7/8/9, CentOS 7/8/9, Rocky Linux 8/9, AlmaLinux 8/9, or Oracle Linux 7/8/9
- SSH access to target hosts with root privileges (or sudo access)
- NetworkManager service must be installed and running on target hosts
- `nmcli` command must be available on target hosts

## Usage

### Step 1: Prepare Inventory File

Create an inventory file (e.g., `inventory`) with your target hosts:

```ini
[rhel]
rhel7.example.com
rhel8.example.com
rhel9.example.com

[rhel7]
Test-RHEL-7.9-1

[net_servers]
server1.example.com
server2.example.com
```

### Step 2: Configure Network Operations (Main Control Panel)

Edit `roles/networkmanager_cli/vars/main.yml` to define your desired network state. This is the **Main Control Panel** where you declare all network operations.

#### Example Configuration

```yaml
---
# roles/networkmanager_cli/vars/main.yml
# 💡 This is the core control file for this Role! Author:FYCheung (fzhang@redhat.com)
# You only need to modify the variables in this file to define all network operations to be executed on target hosts.

# Whether to execute "view network information" operation?
# Set to true to view the output of nmcli dev status and nmcli con show.
nm_action_view: true

# --- Add Network Connections ---
# Define all network connections you want to create in this list.
# Ansible will iterate through this list and create them on target hosts.
nm_connections_to_add:
  # Example 1: Add a static IP address connection
  - name: "static-eth1"      # Network connection name
    ifname: "eth1"            # Physical network interface to bind
    type: "ethernet"          # Connection type
    ip4: "192.168.100.50/24"  # IPv4 address and subnet mask
    gw4: "192.168.100.1"      # IPv4 gateway
    method: "manual"          # IP assignment method: manual means static

  # Example 2: Add a DHCP connection
  - name: "dhcp-eth2"
    ifname: "eth2"
    type: "ethernet"
    method: "auto"            # IP assignment method: auto means DHCP

# --- Modify Network Connections ---
# Define connections and settings you want to modify in this list.
nm_connections_to_modify:
  # Example: Add DNS servers for 'static-eth1' connection
  - name: "static-eth1"
    settings:
      # First time setting DNS
      - { key: "ipv4.dns", value: "114.114.114.114" }
      # Use '+' to append second DNS
      - { key: "+ipv4.dns", value: "8.8.8.8" }

# --- Control Network Connections ---
# Define list of connection names to be activated (up)
nm_connections_to_activate:
  - static-eth1
  - dhcp-eth2

# Define list of physical devices to be deactivated (disconnect)
nm_devices_to_deactivate:
  #- eth3 # If you need to deactivate eth3

# --- Delete Network Connections ---
# Define list of connection names to be deleted
nm_connections_to_delete:
  - old-useless-connection
  - temp-connection
```

### Step 3: Execute with Precision

Use tags for precise, surgical execution:

#### View Network Information (Safe Inspection)

First, inspect the current network status:

```bash
# Use -t view parameter to only run view tasks
ansible-playbook networkmanager_management_site.yml -i inventory -t view
```

This will display:
- All network device status (`nmcli dev status`)
- All network connections (`nmcli con show`)

#### Execute Network Operations

```bash
# Execute add and activate tasks
# Use comma to combine tags, execute both add and control tasks
ansible-playbook networkmanager_management_site.yml -i inventory -t add,control

# Execute cleanup tasks
# Execute delete task separately
ansible-playbook networkmanager_management_site.yml -i inventory -t delete
```

## Tag Reference

| Your Intent                       | Command                                                                                     |
| --------------------------------- | ------------------------------------------------------------------------------------------- |
| **View only, safe inspection**    | `ansible-playbook networkmanager_management_site.yml -i inventory -t view`                |
| **Only create new connections**  | `ansible-playbook networkmanager_management_site.yml -i inventory -t add`                |
| **Only modify existing connections** | `ansible-playbook networkmanager_management_site.yml -i inventory -t modify`          |
| **Only activate/deactivate connections** | `ansible-playbook networkmanager_management_site.yml -i inventory -t control`        |
| **Only delete connections**       | `ansible-playbook networkmanager_management_site.yml -i inventory -t delete`              |
| **Combined operations**           | `ansible-playbook networkmanager_management_site.yml -i inventory -t add,delete,view`     |

## Playbook Structure

### Main Playbook (`networkmanager_management_site.yml`)

Simple playbook that includes the `networkmanager_cli` role:

```yaml
---
- name: Automated NetworkManager Management
  hosts: all
  become: true

  roles:
    - networkmanager_cli
```

### Role Structure (`roles/networkmanager_cli/`)

- **`defaults/main.yml`**: Default variables (can be overridden)
- **`vars/main.yml`**: **Main Control Panel** - Edit this file to configure network operations
- **`tasks/main.yml`**: Intelligent dispatch office - Schedules tasks based on configuration
- **`tasks/view.yml`**: View network information (Information Dashboard)
- **`tasks/add.yml`**: Add network connections (Assembly Line)
- **`tasks/modify.yml`**: Modify network connections (Precision Tuning Station)
- **`tasks/control.yml`**: Control network connections (Start/Stop Console)
- **`tasks/delete.yml`**: Delete network connections (Recycling & Cleanup Station)

## Configuration Examples

### Example 1: Complete Network Setup

```yaml
---
nm_action_view: false

nm_connections_to_add:
  - name: "static-eth1-prod"
    ifname: "eth1"            # ⚠️ Note: Ensure this matches your server's interface name!
    type: "ethernet"
    ip4: "192.168.100.50/24"
    gw4: "192.168.100.1"
    method: "manual"

  - name: "dhcp-eth2-prod"
    ifname: "eth2"            # ⚠️ Note: Ensure this matches your server's interface name!
    type: "ethernet"
    method: "auto"

nm_connections_to_modify: []

nm_connections_to_activate:
  - static-eth1-prod
  - dhcp-eth2-prod

nm_devices_to_deactivate: []

nm_connections_to_delete:
  - old-temp-config
```

### Example 2: View Only (Safe Inspection)

```yaml
---
nm_action_view: true
nm_connections_to_add: []
nm_connections_to_modify: []
nm_connections_to_activate: []
nm_devices_to_deactivate: []
nm_connections_to_delete: []
```

### Example 3: Modify DNS Settings

```yaml
---
nm_action_view: false
nm_connections_to_add: []
nm_connections_to_modify:
  - name: "static-eth1"
    settings:
      # Set DNS (replaces existing)
      - { key: "ipv4.dns", value: "114.114.114.114" }
      # Append additional DNS (use '+' prefix)
      - { key: "+ipv4.dns", value: "8.8.8.8" }
      # Set autoconnect
      - { key: "connection.autoconnect", value: "yes" }
nm_connections_to_activate: []
nm_devices_to_deactivate: []
nm_connections_to_delete: []
```

## Best Practices

- **View Before Change**: Always run with `-t view` first to inspect current state:

  ```bash
  ansible-playbook networkmanager_management_site.yml -i inventory -t view
  ```

- **Use Tags for Precision**: Use tags to execute only the operations you need:

  ```bash
  # Safe: View only
  ansible-playbook networkmanager_management_site.yml -i inventory -t view

  # Add connections
  ansible-playbook networkmanager_management_site.yml -i inventory -t add,control
  ```

- **Verify Interface Names**: Always verify network interface names match your hardware before configuring
- **Test in Non-Production**: Test network configurations in non-production environments first

## Troubleshooting

### "NetworkManager service is not available"

- **Check NetworkManager service status:**
  ```bash
  systemctl status NetworkManager
  ```
- **Install NetworkManager if missing:**
  ```bash
  # RHEL 7
  yum install NetworkManager

  # RHEL 8/9
  dnf install NetworkManager
  ```
- **Start NetworkManager service:**
  ```bash
  systemctl start NetworkManager
  systemctl enable NetworkManager
  ```

### "nmcli command is not available"

- NetworkManager package may not be installed
- Install NetworkManager package (see above)
- Verify `nmcli` is in PATH: `which nmcli`

### Connection creation fails

- **Verify interface name exists:**
  ```bash
  nmcli dev status
  ```
- **Check if connection already exists:**
  ```bash
  nmcli con show
  ```
- **Verify IP address is not in use:**
  ```bash
  ip addr show
  ```

### Connection activation fails

- **Check if connection exists:**
  ```bash
  nmcli con show "connection-name"
  ```
- **Verify interface is available:**
  ```bash
  nmcli dev status
  ```
- **Check for conflicting connections on the same interface**

### Modification fails

- **Verify connection name is correct:**
  ```bash
  nmcli con show
  ```
- **Check setting key syntax:**
  - Use `nmcli con show "connection-name"` to see available settings
  - Use `+` prefix to append (e.g., `+ipv4.dns`)
  - Use `-` prefix to remove (e.g., `-ipv4.dns`)

### Deletion fails

- **Connection may be active:**
  ```bash
  nmcli con down "connection-name"
  ```
- **Connection may not exist** (this is non-fatal, playbook will continue)

## Role Design Philosophy

This solution uses Ansible Role architecture with the following design principles:

1. **Declarative Configuration**: Define desired state in `vars/main.yml`, not step-by-step commands
2. **Modularity**: Clear separation of concerns (view, add, modify, control, delete)
3. **Tag-Based Precision**: Use tags for surgical execution control
4. **Reusability**: Role can be used in any playbook or project
5. **Maintainability**: Easy to update and extend
6. **Team Collaboration**: Standard structure for team development
7. **Configuration Flexibility**: Centralized control panel in `vars/main.yml`

## Author

GCG AAP SSA Team + v3.01 20260217

## License

📜 License Type: End User License Agreement (EULA)
🔒 Authorization: Subscription-based License
