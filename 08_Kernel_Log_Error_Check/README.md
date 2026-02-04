# Automated Kernel Log Error Monitoring

## Overview

This Ansible Role-based solution provides automated kernel log error monitoring across multiple Linux hosts. It proactively detects kernel-level errors, classifies them into Critical and Severe categories, sends immediate alerts, and generates comprehensive HTML reports that are collected back to the control node for centralized analysis.

## Reports
<img width="1867" height="1085" alt="image" src="https://github.com/user-attachments/assets/70e4c6cf-608d-431b-97d9-2b0c2ab41349" />


## Features

- **Role-Based Architecture**: Modular, reusable, and maintainable design
- **OS Compatibility**: Supports RHEL 7/8/9 and CentOS 7/8/9 with automatic log source detection
- **Proactive Monitoring**: Detects kernel errors before they cause system failures
- **Tiered Alerting**: Classifies errors into Critical (immediate wall broadcast) and Severe (logged to file)
- **Environment Adaptive**: Automatically uses `/var/log/messages` for RHEL 7 and `journalctl` for RHEL 8/9
- **Detailed Reports**: Generates comprehensive HTML reports with error classification (typically ~4KB per report)
- **Centralized Collection**: Automatically collects reports back to control node
- **Comprehensive Validation**: Parameter validation and OS compatibility checks
- **Error Handling**: Robust error handling with detailed debugging information
- **Handler-Based Alerts**: Uses Ansible handlers for efficient alert delivery
- **Tested and Verified**: Successfully tested across RHEL 7.9, 8.7, and 9.4 with 100% success rate

## Files

- `kernel_log_monitor_site.yml` - Main playbook file
- `roles/kernel_log_monitor/` - Kernel log monitor role
  - `defaults/main.yml` - Default variables (error patterns, paths)
  - `tasks/main.yml` - Core role tasks
  - `handlers/main.yml` - Alert handlers (wall broadcast, file logging)
  - `templates/kernel_report.html.j2` - HTML report template
- `README.md` - This file

## Prerequisites

- Ansible 2.9 or later
- Ansible control node with network access to target hosts
- Target hosts: RHEL 7/8/9 or CentOS 7/8/9
- SSH access to target hosts with root privileges (or sudo access)
- For RHEL 7: `/var/log/messages` must be readable
- For RHEL 8/9: `journalctl` command must be available

## Usage

### 1. Prepare Inventory File

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

[all:children]
rhel7
rhel8
rhel9
```

**Important Notes**:

- This playbook supports RHEL 7/8/9 and CentOS 7/8/9 only
- You can organize hosts into groups as needed
- The playbook will validate OS compatibility before execution

### 2. Customize Error Patterns (Optional)

Edit `roles/kernel_log_monitor/defaults/main.yml` to customize which kernel errors to monitor:

```yaml
kernel_errors:
  critical:
    - 'Call Trace'
    - 'panic'
    - 'hung_task_timeout_secs'
    # Add more critical error patterns as needed
  severe:
    - 'oom'
    - 'segfault'
    - 'hardware error'
    # Add more severe error patterns as needed
```

**Note**: These are regex patterns that will be matched against kernel log lines (case-insensitive).

### 3. Customize Log Lines and Paths (Optional)

Edit `roles/kernel_log_monitor/defaults/main.yml` to change monitoring configuration:

```yaml
# Number of log lines to capture
log_lines: 100

# Report storage paths
report_base_path: "/tmp"
collection_dest_path: "/tmp"

# Kernel events log file path (for severe errors)
kernel_events_log: "/var/log/kernel_events.log"
```

### 4. Run the Playbook

Execute the playbook from the control node:

```bash
ansible-playbook kernel_log_monitor_site.yml -i inventory
```

### 5. Review Results

After execution, you should see output similar to:

```
PLAY RECAP ******************************************************************
TEST-RHEL-8.9-1            : ok=25   changed=2    failed=0
Test-RHEL-7.9-1            : ok=27   changed=2    failed=0
ansible25.example.com      : ok=25   changed=2    failed=0
localhost                  : ok=2    changed=0    failed=0
```

**Review the results**:

- **Immediate Alerts**: If Critical errors were detected, all logged-in users on affected hosts received a wall broadcast message
- **Log Files**: Severe errors are logged to `/var/log/kernel_events.log` on affected hosts
- **HTML Reports**: Reports are collected to the control node's `/tmp/` directory (or your configured `collection_dest_path`):

```bash
# List all collected reports
ls -lh /tmp/kernel_report_*.html

# Expected output (example):
# -rw-r--r-- 1 root root 4.3K Jan 27 10:30 kernel_report_Test-RHEL-7.9-1.html
# -rw-r--r-- 1 root root 4.3K Jan 27 10:30 kernel_report_TEST-RHEL-8.9-1.html
# -rw-r--r-- 1 root root 4.3K Jan 27 10:30 kernel_report_ansible25.example.com.html

# Open a specific report in a web browser
# On Linux:
xdg-open /tmp/kernel_report_Test-RHEL-7.9-1.html

# On Windows (using WSL or Git Bash):
start /tmp/kernel_report_Test-RHEL-7.9-1.html
```

## Report Format

Each report is an HTML file (typically 4-5KB in size) containing:

1. **Summary Information**: Hostname, OS version, report generation time
2. **Status Indicator**: Visual status (healthy or errors detected)
3. **Critical Errors Section**: List of critical errors that triggered wall broadcast
4. **Severe Warnings Section**: List of severe errors logged to file
5. **Report Information**: Log source, lines analyzed, OS details

The report uses color coding:

- **Green (✅ Healthy)**: No kernel errors detected
- **Red (⚠️ Errors Detected)**: Kernel errors found, requires attention

**Report File Size**: Based on test results, typical report sizes are:

- RHEL 7: ~4,316 bytes
- RHEL 8: ~4,317 bytes
- RHEL 9: ~4,329 bytes

Report size may vary slightly based on detected errors and OS version.

## Playbook Structure

### Main Playbook (`kernel_log_monitor_site.yml`)

The playbook consists of three plays:

1. **Play 1: Execute Kernel Log Monitor**

   - OS compatibility validation
   - Executes `kernel_log_monitor` role on all target hosts
   - Captures kernel logs (RHEL 7: `/var/log/messages`, RHEL 8/9: `journalctl`)
   - Classifies errors and sends alerts
   - Generates HTML reports on each host
2. **Play 2: Collect Reports**

   - Fetches HTML report files from all target hosts
   - Collects reports to control node's `/tmp/` directory
3. **Play 3: Final Summary**

   - Displays completion summary and next steps
   - Runs on localhost

### Role Structure (`roles/kernel_log_monitor/`)

- **`defaults/main.yml`**: Default variables (error patterns, paths, log lines)
- **`tasks/main.yml`**: Core role tasks
  - Parameter validation
  - OS compatibility check
  - Kernel log capture (RHEL 7 vs 8/9)
  - Error classification (Critical vs Severe)
  - Alert triggering (via handlers)
  - HTML report generation
- **`handlers/main.yml`**: Alert handlers
  - `emergency_alert`: Wall broadcast for critical errors
  - `priority_alert`: File logging for severe errors
- **`templates/kernel_report.html.j2`**: HTML report template with styling

## Customization

### Add or Remove Error Patterns

Edit `roles/kernel_log_monitor/defaults/main.yml`:

```yaml
kernel_errors:
  critical:
    - 'Call Trace'
    - 'panic'
    - 'hung_task_timeout_secs'
    - 'kernel BUG'  # Add new critical pattern
  severe:
    - 'oom'
    - 'segfault'
    - 'hardware error'
    - 'I/O error'   # Add new severe pattern
```

### Change Log Lines to Capture

Edit `roles/kernel_log_monitor/defaults/main.yml`:

```yaml
# Capture more log lines for deeper analysis
log_lines: 500
```

### Change Report Storage Location

Edit `roles/kernel_log_monitor/defaults/main.yml`:

```yaml
report_base_path: "/var/log/kernel_monitor"      # On remote hosts
collection_dest_path: "/opt/reports/kernel_monitor"  # On control node
```

### Change Kernel Events Log Path

Edit `roles/kernel_log_monitor/defaults/main.yml`:

```yaml
kernel_events_log: "/var/log/kernel_events.log"
```

### Override Variables at Runtime

You can override variables when running the playbook:

```bash
ansible-playbook kernel_log_monitor_site.yml -i inventory \
  -e "log_lines=500" \
  -e "report_base_path=/var/log/kernel_monitor"
```

## Best Practices

- **Test First**: Use `--check` mode to preview changes:

  ```bash
  ansible-playbook kernel_log_monitor_site.yml -i inventory --check
  ```
- **Use Tags**: Use tags for selective execution:

  ```bash
  # Only run validation
  ansible-playbook kernel_log_monitor_site.yml -i inventory --tags validation

  # Only run monitoring
  ansible-playbook kernel_log_monitor_site.yml -i inventory --tags monitor

  # Only collect reports
  ansible-playbook kernel_log_monitor_site.yml -i inventory --tags collection
  ```
- **Regular Scheduling**: Schedule regular kernel log monitoring using cron or AAP schedules
- **Report Analysis**: Aggregate reports for trend analysis and capacity planning
- **Alert Response**: Establish procedures for responding to Critical and Severe alerts
- **Documentation**: Document kernel error patterns and corrective actions

## Alert Behavior

### Critical Errors

When Critical errors are detected:

- **Wall Broadcast**: All logged-in users on the host receive an immediate broadcast message
- **Console Output**: Errors are displayed in Ansible console output
- **HTML Report**: Errors are highlighted in the generated report

### Severe Errors

When Severe errors are detected:

- **File Logging**: Errors are appended to `/var/log/kernel_events.log` with timestamp
- **Console Output**: Errors are displayed in Ansible console output
- **HTML Report**: Errors are listed in the generated report

## Troubleshooting

### Playbook fails with "Unsupported operating system"

- Verify target host OS is RHEL 7/8/9 or CentOS 7/8/9
- Check `ansible_distribution` and `ansible_distribution_major_version` facts
- Remove any unsupported hosts from the inventory

### "kernel_errors must be defined with critical and severe lists"

- Ensure `kernel_errors` is defined in `defaults/main.yml`
- Verify both `critical` and `severe` lists are present
- Check YAML syntax in `defaults/main.yml`

### "Failed to capture kernel logs"

- **For RHEL 7**: Check if `/var/log/messages` exists and is readable:
  ```bash
  ls -la /var/log/messages
  ```
- **For RHEL 8/9**: Check if `journalctl` command is available:
  ```bash
  which journalctl
  journalctl --dmesg --lines=10
  ```
- **Check permissions**: Ensure the playbook runs with appropriate privileges

### "Wall command failed"

- Wall command may not be available in all environments
- This is a non-fatal error - the playbook will continue
- Critical errors will still be logged and reported

### Reports not collected to control node

- **Check fetch module permissions:**
  ```bash
  ls -la /tmp/  # On control node
  ```
- **Verify source path exists on remote host:**
  ```bash
  ansible all -i inventory -m shell -a "ls -la /tmp/kernel_report_*.html"
  ```
- **Check network connectivity:**
  ```bash
  ansible all -i inventory -m ping
  ```

### No errors detected but system has issues

- **Increase log lines**: Edit `log_lines` in `defaults/main.yml` to capture more history
- **Add error patterns**: Review kernel logs manually and add relevant patterns
- **Check log source**: Verify the correct log source is being used for your OS version

### "Failed to generate HTML report"

- **Check template file exists:**
  ```bash
  ls -la roles/kernel_log_monitor/templates/kernel_report.html.j2
  ```
- **Check file permissions:**
  ```bash
  chmod 644 roles/kernel_log_monitor/templates/kernel_report.html.j2
  ```
- **Verify Jinja2 syntax**: Check template for syntax errors

## Tested and Verified

This playbook has been tested and verified on the following environments:

- **RHEL 7.9**: Tested on 3 hosts (Test-RHEL-7.9-1, Test-RHEL-7.9-2, Test-RHEL-7.9-3)
- **RHEL 8.7**: Tested on 1 host (TEST-RHEL-8.9-1)
- **RHEL 9.4**: Tested on 1 host (ansible25.example.com)

**Test Results Summary**:

- **Total Hosts Tested**: 5
- **Success Rate**: 100% (all hosts completed successfully)
- **Tasks per Host**:
  - RHEL 7: 27 tasks (2 changed, 4 skipped)
  - RHEL 8/9: 25 tasks (2 changed, 6 skipped)
- **Report Generation**: All hosts successfully generated HTML reports
- **Report Collection**: All reports successfully collected to control node
- **No Failures**: Zero failed tasks across all test runs

## Example Output

### Playbook Execution (Actual Test Results)

```
PLAY [Execute kernel log monitor and generate reports on all hosts] **********

TASK [Gathering Facts] ******************************************************
ok: [Test-RHEL-7.9-1]
ok: [Test-RHEL-7.9-2]
ok: [Test-RHEL-7.9-3]
ok: [TEST-RHEL-8.9-1]
ok: [ansible25.example.com]

TASK [Validate OS compatibility (RHEL/CentOS 7/8/9 only)] ******************
ok: [Test-RHEL-7.9-1] => {
    "msg": "OS compatibility check passed: RedHat 7"
}
ok: [TEST-RHEL-8.9-1] => {
    "msg": "OS compatibility check passed: RedHat 8"
}
ok: [ansible25.example.com] => {
    "msg": "OS compatibility check passed: RedHat 9"
}

TASK [kernel_log_monitor : Validate kernel_errors parameter] ****************
ok: [Test-RHEL-7.9-1] => {
    "msg": "Parameter validation passed: 3 critical patterns, 3 severe patterns"
}

TASK [kernel_log_monitor : Capture recent kernel logs (RHEL 7)] ************
ok: [Test-RHEL-7.9-1]

TASK [kernel_log_monitor : Capture recent kernel logs (RHEL 8/9)] **********
ok: [TEST-RHEL-8.9-1]
ok: [ansible25.example.com]

TASK [kernel_log_monitor : Display classification results for debugging] ****
ok: [Test-RHEL-7.9-1] => {
    "msg": [
        "Critical errors detected: 0",
        "Severe errors detected: 0"
    ]
}

TASK [kernel_log_monitor : Display no errors found message] ****************
ok: [Test-RHEL-7.9-1] => {
    "msg": "✅ No kernel errors detected. System is healthy."
}

TASK [kernel_log_monitor : Generate HTML report] ****************************
changed: [Test-RHEL-7.9-1]
changed: [TEST-RHEL-8.9-1]
changed: [ansible25.example.com]

TASK [kernel_log_monitor : Display report generation success] **************
ok: [Test-RHEL-7.9-1] => {
    "msg": [
        "✅ HTML report generated successfully",
        "Report path: /tmp/kernel_report_Test-RHEL-7.9-1.html",
        "Report file exists: True",
        "Report file size: 4316 bytes"
    ]
}

PLAY [Collect kernel log monitor reports from all hosts] *******************

TASK [Fetch report file from remote host] ***********************************
changed: [Test-RHEL-7.9-1]
changed: [TEST-RHEL-8.9-1]
changed: [ansible25.example.com]

PLAY [Display final summary and confirmation] ******************************

TASK [Display completion summary] ******************************************
ok: [localhost] => {
    "msg": [
        "✅ All kernel log monitor reports have been successfully collected!",
        "Reports are stored in the control node's /tmp/ directory.",
        "Report file naming format: kernel_report_<hostname>.html",
        "You can review individual reports in a web browser for detailed analysis."
    ]
}

PLAY RECAP ******************************************************************
TEST-RHEL-8.9-1            : ok=25   changed=2    unreachable=0    failed=0    skipped=6
Test-RHEL-7.9-1            : ok=27   changed=2    unreachable=0    failed=0    skipped=4
Test-RHEL-7.9-2            : ok=27   changed=2    unreachable=0    failed=0    skipped=4
Test-RHEL-7.9-3            : ok=27   changed=2    unreachable=0    failed=0    skipped=4
ansible25.example.com      : ok=25   changed=2    unreachable=0    failed=0    skipped=6
localhost                  : ok=2    changed=0    unreachable=0    failed=0    skipped=0
```

### Performance Metrics

Based on test results:

- **Execution Time**: Typically completes in seconds per host
- **Log Capture**:
  - RHEL 7: Captures 100 lines from `/var/log/messages`
  - RHEL 8/9: Captures 100+ lines from `journalctl --dmesg`
- **Report Generation**: ~4KB HTML reports generated per host
- **Resource Usage**: Minimal - read-only operations on target hosts

## Role Design Philosophy

This solution uses Ansible Role architecture for the following benefits:

1. **Modularity**: Clear separation of concerns (variables vs. tasks vs. handlers vs. templates)
2. **Reusability**: Role can be used in any playbook or project
3. **Maintainability**: Easy to update and extend
4. **Team Collaboration**: Standard structure for team development
5. **Configuration Flexibility**: Variables in `defaults/main.yml` allow easy customization
6. **Handler-Based Alerts**: Efficient alert delivery using Ansible handlers
7. **Proactive Monitoring**: Detects issues before they cause system failures

## Security Considerations

- **Root Access Required**: This playbook requires root privileges to read kernel logs and send wall broadcasts
- **Report Storage**: Reports may contain sensitive information (system errors, hostnames) - secure storage locations
- **Network Access**: Ensure secure network communication between control node and targets
- **File Permissions**: Reports are created with `0644` permissions - adjust as needed for your security requirements
- **Alert Broadcasting**: Wall broadcasts are sent to all logged-in users - ensure appropriate notification procedures
- **Log File Access**: Kernel events log may contain sensitive information - protect accordingly

## Author

GCG AAP SSA Team + v3.01 20260217

## License

📜 License Type: End User License Agreement (EULA)
🔒 Authorization: Subscription-based License

