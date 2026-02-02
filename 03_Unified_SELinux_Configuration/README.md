# SELinux Configuration Management and Validation

## Overview

This playbook manages SELinux runtime and persistent configuration states across multiple RHEL/CentOS hosts. It provides safe, controlled SELinux state management with validation, error handling, and reboot notifications. The playbook does NOT automatically reboot servers, but provides clear notifications when a reboot is required for configuration changes to take full effect.

## output
```yaml
TASK [Output final SELinux status report] *****************************************************************************************************************************************
ok: [Test-RHEL-7.9-1] => {
    "msg": [
        "================= SELinux Status Report ==================",
        "Host: Test-RHEL-7.9-1",
        "Timestamp: 2026-02-02T12:09:42Z",
        "",
        "Runtime Status:",
        "  - Current Mode: disabled",
        "  - Was changed in this run: No",
        "",
        "Persistent Status (in /etc/selinux/config):",
        "  - Configured Mode: disabled",
        "  - Was changed in this run: Yes",
        "  - Reboot Required: Yes",
        "=========================================================="
    ]
}
ok: [Test-RHEL-7.9-2] => {
    "msg": [
        "================= SELinux Status Report ==================",
        "Host: Test-RHEL-7.9-2",
        "Timestamp: 2026-02-02T12:06:12Z",
        "",
        "Runtime Status:",
        "  - Current Mode: disabled",
        "  - Was changed in this run: No",
        "",
        "Persistent Status (in /etc/selinux/config):",
        "  - Configured Mode: disabled",
        "  - Was changed in this run: Yes",
        "  - Reboot Required: Yes",
        "=========================================================="
    ]
}
ok: [Test-RHEL-7.9-3] => {
    "msg": [
        "================= SELinux Status Report ==================",
        "Host: Test-RHEL-7.9-3",
        "Timestamp: 2026-02-02T19:55:56Z",
        "",
        "Runtime Status:",
        "  - Current Mode: disabled",
        "  - Was changed in this run: No",
        "",
        "Persistent Status (in /etc/selinux/config):",
        "  - Configured Mode: disabled",
        "  - Was changed in this run: No",
        "  - Reboot Required: No",
        "=========================================================="
    ]
}
ok: [TEST-RHEL-8.9-1] => {
    "msg": [
        "================= SELinux Status Report ==================",
        "Host: TEST-RHEL-8.9-1",
        "Timestamp: 2026-02-02T12:09:42Z",
        "",
        "Runtime Status:",
        "  - Current Mode: permissive",
        "  - Was changed in this run: Yes",
        "",
        "Persistent Status (in /etc/selinux/config):",
        "  - Configured Mode: disabled",
        "  - Was changed in this run: Yes",
        "  - Reboot Required: Yes",
        "=========================================================="
    ]
}
ok: [ansible25.example.com] => {
    "msg": [
        "================= SELinux Status Report ==================",
        "Host: ansible25.example.com",
        "Timestamp: 2026-02-02T12:09:43Z",
        "",
        "Runtime Status:",
        "  - Current Mode: disabled",
        "  - Was changed in this run: No",
        "",
        "Persistent Status (in /etc/selinux/config):",
        "  - Configured Mode: disabled",
        "  - Was changed in this run: Yes",
        "  - Reboot Required: Yes",
        "=========================================================="
    ]
}

TASK [Display completion summary] *************************************************************************************************************************************************
ok: [Test-RHEL-7.9-1] => {
    "msg": "SELinux configuration management completed for Test-RHEL-7.9-1"
}
ok: [Test-RHEL-7.9-2] => {
    "msg": "SELinux configuration management completed for Test-RHEL-7.9-2"
}
ok: [Test-RHEL-7.9-3] => {
    "msg": "SELinux configuration management completed for Test-RHEL-7.9-3"
}
ok: [TEST-RHEL-8.9-1] => {
    "msg": "SELinux configuration management completed for TEST-RHEL-8.9-1"
}
ok: [ansible25.example.com] => {
    "msg": "SELinux configuration management completed for ansible25.example.com"
}

PLAY RECAP ************************************************************************************************************************************************************************
TEST-RHEL-8.9-1            : ok=25   changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
Test-RHEL-7.9-1            : ok=23   changed=1    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0
Test-RHEL-7.9-2            : ok=23   changed=1    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0
Test-RHEL-7.9-3            : ok=21   changed=0    unreachable=0    failed=0    skipped=4    rescued=0    ignored=0
ansible25.example.com      : ok=23   changed=1    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0

```

## Features

- **OS Compatibility**: Supports RHEL 7/8/9 and CentOS 7/8/9
- **Dual State Management**: Manages both runtime and persistent SELinux states
- **Configuration Validation**: Validates SELinux state values before applying changes
- **Error Handling**: Comprehensive error handling with rollback capabilities
- **Reboot Notifications**: Clear notifications when reboot is required (no automatic reboots)
- **Status Reporting**: Detailed status reports for each host
- **Idempotent**: Safe to run multiple times without side effects

## Files

- `selinux_config_management.yml` - Main playbook file
- `README.md` - This file

## Prerequisites

- Ansible 2.9 or later
- Ansible control node with network access to target hosts
- Target hosts: RHEL 7/8/9 or CentOS 7/8/9
- SSH access to target hosts with root privileges (or sudo access)
- `ansible.posix` collection installed on the control node

### Installing Required Collection

Before running this playbook, ensure the `ansible.posix` collection is installed in the local `collections/` directory.

#### Option 1: Using the Installation Script (Recommended)

We provide an installation script that installs the collection in the same directory as the playbook:

```bash
cd 02_AAP_Starter_Pack/02_Playbooks/Ansible救火热线系列之(3)全网Selinux统一配置与验证
chmod +x install_collection.sh
./install_collection.sh
```

The script will:

- Check if `ansible-galaxy` is available
- Create a `collections/` directory if it doesn't exist
- Install the `ansible.posix` collection to `collections/` directory
- Verify the installation using `ansible-galaxy collection list -p collections/`
- Display all installed collections

#### Option 2: Manual Installation

If you prefer to install manually, use the following command to install the collection in the local `collections/` directory:

```bash
cd 02_AAP_Starter_Pack/02_Playbooks/Ansible救火热线系列之(3)全网Selinux统一配置与验证
ansible-galaxy collection install ansible.posix -p collections/ --force
```

Then verify the installation:

```bash
ansible-galaxy collection list -p collections/ | grep ansible.posix
```

**Important**: The `-p collections/` parameter ensures the collection is installed in the same directory as the playbook, making it available when you run the playbook.

## Usage

### 1. Update Playbook Variables

Edit the playbook to set your desired SELinux states:

```yaml
vars:
  permanent_state: "disabled"    # Options: enforcing, permissive, disabled
  runtime_state: "permissive"    # Options: enforcing, permissive
```

**SELinux State Options:**

- `enforcing`: SELinux is active and enforcing security policies
- `permissive`: SELinux is active but only logs violations (does not enforce)
- `disabled`: SELinux is completely disabled

### 2. Prepare Inventory File

Create an inventory file (e.g., `inventory`) with your target hosts:

```ini
[rhel7]
Test-RHEL-7.9-1
Test-RHEL-7.9-2
Test-RHEL-7.9-3

[rhel8]
TEST-RHEL-8.9-1

[rhel9]
ansible25.example.com
```

### 3. Run the Playbook

Execute the playbook from the control node:

```bash
ansible-playbook selinux_config_management.yml -i inventory
```

### 4. Review Results

After execution, review the output for:

- SELinux status reports for each host
- Configuration change notifications
- Reboot requirements (if any)

## Customization

### Change SELinux States

Modify the `permanent_state` and `runtime_state` variables:

```yaml
vars:
  permanent_state: "enforcing"    # Persistent state after reboot
  runtime_state: "enforcing"     # Immediate runtime state
```

### Change Configuration File Path

Modify the `config_path` variable (default: `/etc/selinux/config`):

```yaml
vars:
  config_path: "/etc/selinux/config"
```

## Understanding SELinux States

### Runtime State vs Persistent State

- **Runtime State**: The current active SELinux mode (can be changed immediately)
- **Persistent State**: The SELinux mode that will be active after system reboot

### State Transitions

| Current State           | Target State | Reboot Required  | Notes |
| ----------------------- | ------------ | ---------------- | ----- |
| enforcing → permissive | No           | Immediate change |       |
| permissive → enforcing | No           | Immediate change |       |
| enforcing → disabled   | Yes          | Requires reboot  |       |
| permissive → disabled  | Yes          | Requires reboot  |       |
| disabled → enforcing   | Yes          | Requires reboot  |       |
| disabled → permissive  | Yes          | Requires reboot  |       |

## Playbook Structure

1. **OS Compatibility Validation**: Ensures only supported operating systems are processed
2. **Input Parameter Validation**: Validates SELinux state values and configuration paths
3. **SELinux Availability Check**: Verifies SELinux is available on the system
4. **SELinux Configuration Management**: Updates runtime and persistent states with error handling
5. **Reboot Notification**: Displays clear notifications when reboot is required
6. **Status Reporting**: Generates detailed status reports for each host
7. **Debug Output**: Multiple debug tasks for troubleshooting

## Reboot Requirements

### When Reboot is Required

A reboot is required when the persistent configuration (`/etc/selinux/config`) is changed. The playbook will display a clear notification:

```
===========================================================
[NOTICE] SELinux persistent configuration has been changed.
Server requires a reboot for the new configuration 'disabled' to take full effect.
Please perform a manual reboot during a scheduled maintenance window.
===========================================================
```

### Manual Reboot

The playbook does NOT automatically reboot servers. You must manually reboot during a scheduled maintenance window:

```bash
# On target host
sudo reboot
```

Or use Ansible to reboot (separate playbook or task):

```yaml
- name: Reboot server
  ansible.builtin.reboot:
    msg: "Rebooting to apply SELinux configuration changes"
    reboot_timeout: 300
```

## Best Practices

- **Test First**: Run in check mode first: `ansible-playbook selinux_config_management.yml -i inventory --check`
- **Maintenance Windows**: Schedule reboots during maintenance windows when persistent state changes
- **Use Tags**: Use tags for selective execution:
  - `--tags validation` - Run only validation tasks
  - `--tags runtime` - Run only runtime state changes
  - `--tags config` - Run only persistent configuration changes
  - `--tags report` - Display status reports only
- **Document Changes**: Keep track of SELinux state changes for audit purposes
- **Security Considerations**: Be cautious when disabling SELinux in production environments

## Troubleshooting

### Playbook fails with "Unsupported operating system"

- Verify target host OS is RHEL 7/8/9 or CentOS 7/8/9
- Check `ansible_distribution` and `ansible_distribution_major_version` facts

### "SELinux environment cannot be identified"

- SELinux may not be installed on the target host
- Check if SELinux packages are installed: `rpm -qa | grep selinux`
- Install SELinux packages if needed: `yum install -y libselinux-utils`

### "ansible.posix.selinux module not found"

- Install the `ansible.posix` collection: `ansible-galaxy collection install ansible.posix`
- Verify collection is installed: `ansible-galaxy collection list`

### Configuration validation fails

- Verify SELinux state values are correct: `enforcing`, `permissive`, or `disabled`
- Check configuration file path is correct
- Ensure `/usr/sbin/sestatus` command is available on target hosts

### Runtime state change fails

- Verify SELinux is enabled on the target host
- Check if current mode matches target mode (no change needed)
- Review playbook output for specific error messages

### Persistent configuration not taking effect

- Verify configuration file was updated: `cat /etc/selinux/config`
- Check if reboot is required (playbook will notify)
- Ensure system was rebooted after configuration change

## Security Considerations

- **Root Access Required**: This playbook requires root privileges to modify SELinux configuration
- **Security Impact**: Disabling SELinux reduces system security. Use with caution in production environments
- **Audit Trail**: All configuration changes are logged in Ansible output
- **Backup**: Configuration files are automatically backed up before modification

## Example Output

### When Configuration Changes

```
===========================================================
[NOTICE] SELinux persistent configuration has been changed.
Server requires a reboot for the new configuration 'disabled' to take full effect.
Please perform a manual reboot during a scheduled maintenance window.
===========================================================

================= SELinux Status Report ==================
Host: Test-RHEL-7.9-1
Timestamp: 2023-10-27T12:00:00Z

Runtime Status:
  - Current Mode: permissive
  - Was changed in this run: Yes

Persistent Status (in /etc/selinux/config):
  - Configured Mode: disabled
  - Was changed in this run: Yes
  - Reboot Required: Yes
==========================================================
```

### When No Changes Needed

```
================= SELinux Status Report ==================
Host: TEST-RHEL-8.9-1
Timestamp: 2023-10-27T12:00:00Z

Runtime Status:
  - Current Mode: permissive
  - Was changed in this run: No

Persistent Status (in /etc/selinux/config):
  - Configured Mode: disabled
  - Was changed in this run: No
  - Reboot Required: No
==========================================================
```

## Author

GCG AAP SSA Team + v3.01 20260217

## License

📜 License Type: End User License Agreement (EULA)
🔒 Authorization: Subscription-based License
