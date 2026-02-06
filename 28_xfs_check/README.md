# Automated XFS Filesystem Health Check

## Overview

This playbook automates XFS filesystem health inspection on RHEL/CentOS systems. It performs comprehensive health checks including kernel error analysis, metadata consistency verification, and storage usage statistics collection. The playbook generates detailed inspection reports that are automatically collected back to the control node for centralized analysis and monitoring.

The playbook is designed to be safe for production environments, performing read-only checks that do not modify filesystem data or require filesystem unmounting.

## Features

- **Complete Automation**: End-to-end XFS health inspection from data collection to report generation
- **OS Compatibility**: Supports RHEL 7/8/9 and CentOS 7/8/9 with automatic validation
- **Read-Only Safety**: All filesystem checks are read-only and safe for production systems
- **Comprehensive Reporting**: Generates detailed text reports with kernel logs, metadata status, and usage statistics
- **Centralized Collection**: Automatically collects reports back to control node for centralized analysis
- **Robust Error Handling**: Comprehensive error handling with block-rescue structures ensures execution continues even if individual tasks fail
- **Idempotent Design**: Safe to run multiple times without side effects
- **Comprehensive Validation**: OS compatibility checks and prerequisite validation
- **Multiple Check Types**: Kernel log analysis, metadata consistency checks, and storage usage monitoring

## Prerequisites

### General Prerequisites

- Ansible 2.9 or later
- Ansible control node with network access to target hosts
- Target hosts: RHEL 7/8/9 or CentOS 7/8/9
- SSH access to target hosts (password or key-based authentication)
- Sudo/root privileges on target hosts
- XFS filesystems present on target hosts

### Required Tools on Target Hosts

The playbook requires the following tools to be available on target hosts:

- **xfs_db**: For metadata consistency checks (part of `xfsprogs` package)
- **xfs_repair**: For reference (part of `xfsprogs` package)
- **dmesg**: For kernel log analysis (usually pre-installed)
- **df**: For storage and inode usage collection (usually pre-installed)

If these tools are not available, the playbook will attempt to continue with available tools and skip unavailable checks.

### Package Installation (if needed)

If XFS tools are missing, install them:

```bash
# RHEL 7
yum install -y xfsprogs

# RHEL 8/9
dnf install -y xfsprogs
```

## Files

- `xfs_health_check_site.yml` - Main XFS health inspection playbook
- `inventory` - Host inventory file
- `ansible.cfg` - Ansible configuration file
- `README.md` - This file

## Usage

### 1. Prepare Inventory File

Edit the `inventory` file to define your target hosts. The playbook uses the `all` hosts group by default, but you can use specific groups:

```ini
[xfs_servers]
# Define target hosts for XFS health check here
# Example:
# xfs-server-01 ansible_host=192.168.1.100
# xfs-server-02 ansible_host=192.168.1.101

# Using the standard inventory format:
[rhel7]
Test-RHEL-7.9-1
Test-RHEL-7.9-2
Test-RHEL-7.9-3

[rhel8]
TEST-RHEL-8.9-1

[rhel9]
ansible25.example.com

# Group all RHEL hosts as xfs_servers
[xfs_servers:children]
rhel7
rhel8
rhel9
```

### 2. Verify Prerequisites

Before running the playbook, verify prerequisites on target hosts:

```bash
# Test connectivity
ansible all -i inventory -m ping

# Check if XFS filesystems exist
ansible all -i inventory -m shell -a "mount | grep xfs"

# Verify XFS tools availability
ansible all -i inventory -m shell -a "which xfs_db"
```

### 3. Run the Playbook

#### Basic Execution

```bash
# Run health check on all hosts
ansible-playbook xfs_health_check_site.yml -i inventory
```

#### Check Specific Host or Group

```bash
# Check only RHEL 9 hosts
ansible-playbook xfs_health_check_site.yml -i inventory --limit rhel9

# Check a specific host
ansible-playbook xfs_health_check_site.yml -i inventory --limit Test-RHEL-7.9-1
```

#### Dry Run (Check Mode)

```bash
# Preview what would be done (check mode)
ansible-playbook xfs_health_check_site.yml -i inventory --check
```

#### Advanced Usage

```bash
# Run with increased parallelism for bulk execution
ansible-playbook xfs_health_check_site.yml -i inventory --forks 10

# Run with verbose output for troubleshooting
ansible-playbook xfs_health_check_site.yml -i inventory -v

# Run with extra verbose output
ansible-playbook xfs_health_check_site.yml -i inventory -vvv
```

### 4. Review Reports

After successful execution, review the generated reports:

```bash
# List reports on control node
ls -lh reports/

# View a specific report
cat reports/xfs_health_report_<hostname>.txt

# View reports for all hosts
for report in reports/*.txt; do
  echo "=== $report ==="
  cat "$report"
  echo ""
done
```

## What Gets Configured

The playbook performs the following tasks:

1. **OS Compatibility Validation**: Verifies RHEL/CentOS 7/8/9 compatibility
2. **Prerequisites Validation**: Checks for required XFS tools availability
3. **Report Initialization**: Creates report file with header information
4. **XFS Mount Point Collection**: Identifies all XFS filesystems on the system
5. **Kernel Log Analysis**: Checks dmesg for XFS-related errors (last 20 entries)
6. **Metadata Consistency Check**: Performs read-only metadata checks using xfs_db
7. **Storage Usage Collection**: Collects disk space and inode usage statistics
8. **Report Compilation**: Compiles all collected data into comprehensive report
9. **Report Collection**: Fetches reports back to control node for centralized analysis
10. **Summary Display**: Displays execution summary with key statistics

## Configuration Details

### Report File Location

- **On Target Host**: `/tmp/xfs_health_report_<hostname>.txt`
- **On Control Node**: `./reports/xfs_health_report_<hostname>.txt`

### Customizing Report Path

You can customize the report path by overriding variables:

```bash
ansible-playbook xfs_health_check_site.yml -i inventory \
  -e "report_path=/var/log/xfs_health_report_{{ ansible_hostname }}.txt" \
  -e "report_dest_path=./custom_reports/"
```

### Metadata Check Limitations

**Important**: The `xfs_db` check performed in this playbook runs in read-only mode (`-r` flag) on mounted filesystems. For comprehensive metadata consistency checks:

- **Best Practice**: Unmount the filesystem and run `xfs_repair -n` (dry-run) in maintenance mode
- **Current Implementation**: Uses `xfs_db -r` which may not detect all issues on live filesystems
- **Production Recommendation**: Schedule periodic maintenance windows for full filesystem checks

## Troubleshooting

### Playbook fails with "Unsupported operating system"

- Verify target host OS is RHEL 7/8/9 or CentOS 7/8/9
- Check `ansible_distribution` and `ansible_distribution_major_version` facts
- Remove any unsupported hosts from the inventory

### "No XFS filesystems found on this host"

- Verify XFS filesystems exist: `mount | grep xfs`
- Check if filesystems are properly mounted
- Ensure `ansible_mounts` fact is being collected (gather_facts: true)

### "xfs_db command not available"

- Install xfsprogs package:
  ```bash
  yum install -y xfsprogs  # RHEL 7
  dnf install -y xfsprogs  # RHEL 8/9
  ```
- The playbook will continue with other checks if xfs_db is unavailable

### "Metadata check passed or not supported in live mode"

This is expected behavior. The `xfs_db -r` check on mounted filesystems has limitations:

- **For comprehensive checks**: Unmount filesystem and run `xfs_repair -n` in maintenance mode
- **For production monitoring**: This playbook provides basic health indicators
- **For detailed analysis**: Schedule maintenance windows for offline checks

### Report file was not created successfully

- **Check disk space on target host:**
  ```bash
  df -h /tmp
  ```
- **Check file permissions:**
  ```bash
  ls -la /tmp/xfs_health_report_*.txt
  ```
- **Check Ansible logs for errors:**
  ```bash
  ansible-playbook xfs_health_check_site.yml -i inventory -v
  ```

### Reports not collected to control node

- **Check fetch module permissions:**
  ```bash
  ls -la reports/  # On control node
  ```
- **Verify source path exists on remote host:**
  ```bash
  ansible all -i inventory -m shell -a "ls -la /tmp/xfs_health_report_*.txt"
  ```
- **Check network connectivity:**
  ```bash
  ansible all -i inventory -m ping
  ```
- **Ensure reports directory exists:**
  ```bash
  mkdir -p reports/
  ```

### Kernel log check shows no output

- This is normal if no XFS-related errors are present in dmesg
- To view full dmesg output: `dmesg | grep -i xfs`
- Check system logs: `journalctl -k | grep -i xfs`

## Best Practices

1. **Regular Monitoring**: Schedule regular health checks (e.g., weekly or monthly):
   ```bash
   # Add to crontab
   0 2 * * 0 ansible-playbook /path/to/xfs_health_check_site.yml -i /path/to/inventory
   ```

2. **Maintenance Windows**: Plan periodic maintenance windows for comprehensive offline checks:
   ```bash
   # Unmount filesystem and run full check
   umount /mount/point
   xfs_repair -n /dev/device
   mount /mount/point
   ```

3. **Report Archival**: Archive reports for historical analysis:
   ```bash
   # Archive reports with timestamp
   tar -czf xfs_reports_$(date +%Y%m%d).tar.gz reports/
   ```

4. **Alerting Integration**: Integrate with monitoring systems to alert on critical issues:
   - Parse reports for error patterns
   - Set up alerts for high inode usage (>80%)
   - Monitor disk space thresholds

5. **Baseline Establishment**: Run initial checks to establish baseline health metrics

6. **Documentation**: Keep records of health check results and any remediation actions taken

7. **Testing**: Test playbook on non-production systems first

8. **Backup Before Maintenance**: Always backup critical data before performing filesystem repairs

## Limitations

- **Read-Only Checks**: Metadata checks are performed in read-only mode on mounted filesystems, which may not detect all issues
- **Live Filesystem Limitations**: Some checks require unmounting filesystems for full reliability
- **Tool Dependencies**: Requires xfsprogs package to be installed for comprehensive checks
- **Network Dependency**: Requires network connectivity between control node and target hosts
- **Storage Requirements**: Reports consume disk space on both target hosts and control node
- **Kernel Log Scope**: Only checks last 20 XFS-related entries in dmesg
- **No Automatic Repair**: This playbook only performs checks and reporting; repairs must be performed manually

## Security Considerations

1. **Credential Management**: Use Ansible Vault for sensitive credentials if needed
2. **Report Sensitivity**: Health reports may contain system information; protect report files appropriately
3. **Access Control**: Limit access to reports directory on control node
4. **Audit Trail**: Keep logs of all health check executions for compliance
5. **Network Security**: Ensure secure communication channels between control node and target hosts

## Support

For issues or questions:

- Check Ansible logs: `ansible-playbook -v` for verbose output
- Review system logs: `journalctl -xe` on target hosts
- Verify XFS tools: `which xfs_db` and `which xfs_repair`
- Check Red Hat documentation: https://access.redhat.com/documentation/

## Example Report Output

```
XFS Health Inspection Report
==========================================
Generated on: 2026-02-06T12:00:00Z
Host: test-server-01
OS: RedHat 8.9
==========================================

==========================================
Kernel Log Analysis (dmesg - Last 20 XFS entries)
==========================================
No XFS errors found in dmesg.

==========================================
Filesystem: /dev/sda1
Mount Point: /
==========================================
--- Metadata Consistency Check (xfs_db) ---
Metadata check passed or not supported in live mode. For comprehensive checks, unmount filesystem and run xfs_repair -n.
--- Storage and Inode Usage ---
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1        50G   20G   30G  40% /
Filesystem      Inodes IUsed IFree IUse% Mounted on
/dev/sda1      2621440 50000 2571440    2% /
==========================================

==========================================
End of Report
==========================================
```

## License

📜 License Type: End User License Agreement (EULA)
🔒 Authorization: Subscription-based License

## Author

GCG AAP SSA Team + v3.01 20260217

