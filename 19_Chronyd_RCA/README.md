 ## Chronyd Service Fault Automatic Diagnosis (RHEL7/8/9 & CentOS7/8/9)
 
 ### 1. Overview
 
 This scenario provides a **non-intrusive, read-only** chronyd (chrony) time synchronization health check for RHEL/CentOS 7/8/9.
 The playbook automatically collects key diagnostic information and generates a **structured text report** on each target host.
 
 Main capabilities:
 
 - **OS compatibility assertion**: Only runs on RHEL/CentOS 7/8/9. Unsupported systems are rejected with a clear message.
 - **Package & systemd status**: Checks chrony package installation and chronyd service active/enabled state.
 - **Chrony time status**: Collects `chronyc sources`, `tracking`, `sourcestats` and `activity` outputs.
 - **System time configuration**: Captures `timedatectl status` and `timedatectl show`.
 - **Configuration & directories**: Verifies `chrony.conf` existence and checks log/runtime directory permissions.
 - **Network & firewall diagnostics**: Tests connectivity to the configured NTP server and collects firewalld state/services/ports.
 - **Logs & journal analysis**: Collects recent `journalctl` entries for the chronyd service.
 - **Final report generation**: Renders a consolidated diagnosis report from a Jinja2 template.
 
 The design follows Red Hat Ansible Automation Platform best practices:
 
 - Uses fully qualified module names (e.g. `ansible.builtin.command`).
 - Uses idempotent, read-only checks by default.
 - Uses `debug` tasks after each functional section for easier troubleshooting.
 - Uses `block`/`rescue` to safely handle optional service start attempts.

 ### screenshots
 <img width="2510" height="1319" alt="image" src="https://github.com/user-attachments/assets/5507fffa-7783-4a41-b3c5-ac4ea4cf7c10" />
 <img width="2510" height="1324" alt="image" src="https://github.com/user-attachments/assets/1a0b4579-a672-4149-a41c-45a012d03d5d" />


 ---
 
 ### 2. File Layout
 
 All files for this scenario are under this directory:
 
 - `ansible.cfg` – Local Ansible configuration (inventory, SSH options, fact caching, etc.).
 - `inventory` – Example inventory file.
 - `group_vars/all.yml` – Global variables for this scenario.
 - `chronyd_diagnosis_site.yml` – Main playbook.
 - `templates/chronyd_diagnosis_report.j2` – Jinja2 template used to generate the diagnosis report.
 - `README.md` – This document.
 
 ---
 
 ### 3. Inventory Example (Using Your Standard Groups)
 
 This scenario can be used with your standard inventory groups from the development guide:
 
 ```ini
 [time]
 # Add your target RHEL/CentOS 7/8/9 time synchronization hosts here
 # Example:
 # time1.example.com
 # 10.66.208.232
 Test-RHEL-7.9-1
 
 # Using the standard inventory format from development guide:
 [rhel7]
 Test-RHEL-7.9-1
 Test-RHEL-7.9-2
 Test-RHEL-7.9-3
 
 [rhel8]
 TEST-RHEL-8.9-1
 
 [rhel9]
 ansible25.example.com
 ```
 
 For the playbook, the **target group** is controlled by the `target_group` variable in `group_vars/all.yml`.
 By default it is set to `time`, but you can easily switch it to any group above, for example:
 
 ```yaml
 target_group: rhel9
 ```
 
 ---
 
 ### 4. Variables (group_vars/all.yml)
 
 Key variables you can adjust in `group_vars/all.yml`:
 
 - **Target group**
 
   - `target_group`: Host group name used by the playbook (must exist in `inventory`).
 - **Service and package**
 
   - `chronyd_service`: Service name (default: `chronyd`).
   - `chrony_package`: Package name (default: `chrony`).
 - **NTP server**
 
   - `ntp_server`: Upstream NTP server or pool used for connectivity tests (default: `pool.ntp.org`).
 - **Paths and directories**
 
   - `chrony_conf_path`: Path to the chrony configuration file (default: `/etc/chrony.conf`).
   - `chrony_log_dir`: Chrony log directory (default: `/var/log/chrony`).
   - `chrony_run_dir`: Chrony runtime directory (default: `/run/chrony`).
 - **Logs and report**
 
   - `collect_journal_lines`: Number of lines to collect from `journalctl` (default: `200`).
   - `report_output_path`: Final report path on each host (default: `/root/chronyd_diagnosis_report.txt`).
 - **Optional service start**
 
   - `attempt_service_start`: When `true`, the playbook will **attempt to start** the chronyd service.
     This is wrapped in a `block/rescue` so any failure is captured but **does not stop** the rest of the diagnostics.
 
 ---
 
 ### 5. How to Run
 
 From this directory, you can run the playbook in several ways.
 
 **Basic diagnosis (analysis only, no changes):**
 
 ```bash
 ansible-playbook -i inventory chronyd_diagnosis_site.yml
 ```
 
 **Attempt to start chronyd while diagnosing (for deeper failure analysis):**
 
 ```bash
 ansible-playbook -i inventory chronyd_diagnosis_site.yml -e attempt_service_start=true
 ```
 
 **Run against a specific OS group from your standard inventory:**
 
 ```bash
 # Example: diagnose only RHEL 9 hosts
 ansible-playbook -i inventory chronyd_diagnosis_site.yml -e target_group=rhel9
 ```
 
 ---
 
 ### 6. What You Get
 
 After the playbook finishes, each target host will have a text report similar to:
 
 - Path: `/root/chronyd_diagnosis_report.txt` (configurable via `report_output_path`)
 - Content:
   - OS and kernel version
   - chrony package and systemd service status
   - `chronyc sources`, `tracking`, `sourcestats` and `activity` outputs
   - `timedatectl status` and `timedatectl show` outputs
   - `chrony.conf` presence and basic configuration validation
   - Log/runtime directory permissions
   - NTP server ping results and firewalld configuration
   - Recent `journalctl` entries for the chronyd service
   - A short list of common troubleshooting hints
 
 You can archive these reports centrally or compare them across multiple runs for trend analysis and proactive troubleshooting.
 
 ---
 
 ### 7. Notes and Best Practices
 
 - This playbook is designed to be **safe for production**: by default it runs in analysis-only mode and does not change system configuration.
 - All checks are **non-fatal** wherever possible, so a failure in one area (e.g. missing `chronyc`) will not stop the rest of the diagnostics.
 - Use the `debug` output and the generated report together to quickly pinpoint time synchronization issues across multiple hosts.
 


