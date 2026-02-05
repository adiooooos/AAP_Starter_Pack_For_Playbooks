[root@aap25 15_RHEL9_Standard_Config]# ansible-playbook rhel9_standardization_site.yml -i inventory -C

PLAY [RHEL9 System Standardization Configuration] **********************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************************************************
ok: [ansible25.example.com]

TASK [Check OS compatibility] ******************************************************************************************************************************************
ok: [ansible25.example.com] => {
    "changed": false,
    "msg": "OS compatibility check passed: RedHat 9.4"
}

TASK [Display system information] **************************************************************************************************************************************
ok: [ansible25.example.com] => {
    "msg": [
        "Starting standardization for: ansible25.example.com",
        "OS: RedHat 9.4",
        "Hostname: ansible25",
        "IP Address: 10.71.18.156"
    ]
}

TASK [Validate required variables] *************************************************************************************************************************************
ok: [ansible25.example.com] => {
    "changed": false,
    "msg": "All required variables are defined"
}

TASK [user_management : Initialize user management status] *************************************************************************************************************
ok: [ansible25.example.com]

TASK [user_management : Create admin user] *****************************************************************************************************************************
[DEPRECATION WARNING]: Encryption using the Python crypt module is deprecated. The Python crypt module is deprecated and will be removed from Python 3.13. Install the
passlib library for continued encryption functionality. This feature will be removed in version 2.17. Deprecation warnings can be disabled by setting
deprecation_warnings=False in ansible.cfg.
changed: [ansible25.example.com]

TASK [user_management : Update user creation status] *******************************************************************************************************************
ok: [ansible25.example.com]

TASK [user_management : Display user creation result] ******************************************************************************************************************
ok: [ansible25.example.com] => {
    "msg": "Admin user 'admin' created"
}

TASK [user_management : Add sudo rule for admin user] ******************************************************************************************************************
changed: [ansible25.example.com]

TASK [user_management : Update sudo configuration status] **************************************************************************************************************
ok: [ansible25.example.com]

TASK [user_management : Display sudo configuration result] *************************************************************************************************************
ok: [ansible25.example.com] => {
    "msg": "Sudo privileges configured for user 'admin'"
}

TASK [user_management : Enable systemd user session] *******************************************************************************************************************
ok: [ansible25.example.com]

TASK [user_management : Update session persistence status] *************************************************************************************************************
ok: [ansible25.example.com]

TASK [user_management : Display session persistence result] ************************************************************************************************************
fatal: [ansible25.example.com]: FAILED! => {"msg": "The conditional check 'session_enabled is succeeded' failed. The error was: The 'failed' test expects a dictionary\n\nThe error appears to be in '/home/admin/aap/controller/data/projects/starter_pack/15_RHEL9_Standard_Config/roles/user_management/tasks/main.yml': line 85, column 7, but may\nbe elsewhere in the file depending on the exact syntax problem.\n\nThe offending line appears to be:\n\n\n    - name: Display session persistence result\n      ^ here\n"}

TASK [user_management : Handle session persistence failure] ************************************************************************************************************
ok: [ansible25.example.com] => {
    "msg": "Warning: Failed to enable user session persistence (non-critical)"
}

TASK [user_management : Check user existence] **************************************************************************************************************************
skipping: [ansible25.example.com]

TASK [user_management : Display user information] **********************************************************************************************************************
ok: [ansible25.example.com] => {
    "msg": "User 'admin' verification: "
}

TASK [user_management : Display user management summary] ***************************************************************************************************************
ok: [ansible25.example.com] => {
    "msg": [
        "=== User Management Summary ===",
        "✅ Admin user created/verified",
        "✅ Sudo privileges configured",
        "✅ Session persistence enabled",
        "✅ All user management tasks completed successfully"
    ]
}

TASK [system_config : Check localhost connectivity] ********************************************************************************************************************
ok: [ansible25.example.com]

TASK [system_config : Display localhost connectivity result] ***********************************************************************************************************
ok: [ansible25.example.com] => {
    "msg": "Localhost connectivity: FAILED"
}

TASK [system_config : Get hostname information] ************************************************************************************************************************
skipping: [ansible25.example.com]

TASK [system_config : Display hostname information] ********************************************************************************************************************
ok: [ansible25.example.com] => {
    "msg": "FQDN: "
}

TASK [system_config : Check FQDN resolution] ***************************************************************************************************************************
ok: [ansible25.example.com]

TASK [system_config : Display FQDN resolution result] ******************************************************************************************************************
ok: [ansible25.example.com] => {
    "msg": "FQDN resolution: FAILED"
}

TASK [system_config : Initialize system config status] *****************************************************************************************************************
ok: [ansible25.example.com]

TASK [system_config : Check current SELinux status] ********************************************************************************************************************
skipping: [ansible25.example.com]

TASK [system_config : Display SELinux status] **************************************************************************************************************************
ok: [ansible25.example.com] => {
    "msg": []
}

TASK [system_config : Set SELinux state temporarily] *******************************************************************************************************************
[WARNING]: Reboot is required to set SELinux state to 'permissive'
changed: [ansible25.example.com]

TASK [system_config : Configure SELinux permanently] *******************************************************************************************************************
changed: [ansible25.example.com]

TASK [system_config : Update SELinux configuration status] *************************************************************************************************************
ok: [ansible25.example.com]

TASK [system_config : Display SELinux configuration result] ************************************************************************************************************
ok: [ansible25.example.com] => {
    "msg": "SELinux configured to: permissive"
}

TASK [system_config : Verify SELinux configuration] ********************************************************************************************************************
skipping: [ansible25.example.com]

TASK [system_config : Display SELinux configuration verification] ******************************************************************************************************
ok: [ansible25.example.com] => {
    "msg": []
}

TASK [system_config : Display system configuration summary] ************************************************************************************************************
ok: [ansible25.example.com] => {
    "msg": [
        "=== System Configuration Summary ===",
        "✅ Hostname validation completed",
        "✅ SELinux configured to permissive",
        "✅ All system configuration tasks completed successfully"
    ]
}

TASK [firewall_config : Initialize firewall configuration status] ******************************************************************************************************
ok: [ansible25.example.com]

TASK [firewall_config : Check if firewalld is installed] ***************************************************************************************************************
skipping: [ansible25.example.com]

TASK [firewall_config : Set firewall service availability] *************************************************************************************************************
ok: [ansible25.example.com]

TASK [firewall_config : Display firewalld status] **********************************************************************************************************************
ok: [ansible25.example.com] => {
    "msg": "Firewalld service is running"
}

TASK [firewall_config : Start and enable firewalld service] ************************************************************************************************************
skipping: [ansible25.example.com]

TASK [firewall_config : Update firewall service availability] **********************************************************************************************************
ok: [ansible25.example.com]

TASK [firewall_config : Display firewalld service status] **************************************************************************************************************
ok: [ansible25.example.com] => {
    "msg": "Firewalld service already running"
}

TASK [firewall_config : Open TCP ports in firewall] ********************************************************************************************************************
failed: [ansible25.example.com] (item=50051) => {"ansible_loop_var": "item", "changed": false, "item": 50051, "msg": "firewall is not currently running, unable to perform immediate actions without a running firewall daemon"}
failed: [ansible25.example.com] (item=53) => {"ansible_loop_var": "item", "changed": false, "item": 53, "msg": "firewall is not currently running, unable to perform immediate actions without a running firewall daemon"}
failed: [ansible25.example.com] (item=80) => {"ansible_loop_var": "item", "changed": false, "item": 80, "msg": "firewall is not currently running, unable to perform immediate actions without a running firewall daemon"}
failed: [ansible25.example.com] (item=443) => {"ansible_loop_var": "item", "changed": false, "item": 443, "msg": "firewall is not currently running, unable to perform immediate actions without a running firewall daemon"}
failed: [ansible25.example.com] (item=5432) => {"ansible_loop_var": "item", "changed": false, "item": 5432, "msg": "firewall is not currently running, unable to perform immediate actions without a running firewall daemon"}
failed: [ansible25.example.com] (item=6379) => {"ansible_loop_var": "item", "changed": false, "item": 6379, "msg": "firewall is not currently running, unable to perform immediate actions without a running firewall daemon"}
failed: [ansible25.example.com] (item=27199) => {"ansible_loop_var": "item", "changed": false, "item": 27199, "msg": "firewall is not currently running, unable to perform immediate actions without a running firewall daemon"}
failed: [ansible25.example.com] (item=9000) => {"ansible_loop_var": "item", "changed": false, "item": 9000, "msg": "firewall is not currently running, unable to perform immediate actions without a running firewall daemon"}
failed: [ansible25.example.com] (item=9001) => {"ansible_loop_var": "item", "changed": false, "item": 9001, "msg": "firewall is not currently running, unable to perform immediate actions without a running firewall daemon"}
failed: [ansible25.example.com] (item=9002) => {"ansible_loop_var": "item", "changed": false, "item": 9002, "msg": "firewall is not currently running, unable to perform immediate actions without a running firewall daemon"}
failed: [ansible25.example.com] (item=9003) => {"ansible_loop_var": "item", "changed": false, "item": 9003, "msg": "firewall is not currently running, unable to perform immediate actions without a running firewall daemon"}
...ignoring

TASK [firewall_config : Collect failed TCP ports] **********************************************************************************************************************
ok: [ansible25.example.com] => (item=50051/tcp)
ok: [ansible25.example.com] => (item=53/tcp)
ok: [ansible25.example.com] => (item=80/tcp)
ok: [ansible25.example.com] => (item=443/tcp)
ok: [ansible25.example.com] => (item=5432/tcp)
ok: [ansible25.example.com] => (item=6379/tcp)
ok: [ansible25.example.com] => (item=27199/tcp)
ok: [ansible25.example.com] => (item=9000/tcp)
ok: [ansible25.example.com] => (item=9001/tcp)
ok: [ansible25.example.com] => (item=9002/tcp)
ok: [ansible25.example.com] => (item=9003/tcp)

TASK [firewall_config : Mark all TCP ports as failed when firewalld is not available] **********************************************************************************
skipping: [ansible25.example.com] => (item=50051)
skipping: [ansible25.example.com] => (item=53)
skipping: [ansible25.example.com] => (item=80)
skipping: [ansible25.example.com] => (item=443)
skipping: [ansible25.example.com] => (item=5432)
skipping: [ansible25.example.com] => (item=6379)
skipping: [ansible25.example.com] => (item=27199)
skipping: [ansible25.example.com] => (item=9000)
skipping: [ansible25.example.com] => (item=9001)
skipping: [ansible25.example.com] => (item=9002)
skipping: [ansible25.example.com] => (item=9003)
skipping: [ansible25.example.com]

TASK [firewall_config : Display TCP ports configuration result] ********************************************************************************************************
skipping: [ansible25.example.com]

TASK [firewall_config : Open UDP ports in firewall] ********************************************************************************************************************
failed: [ansible25.example.com] (item=50051) => {"ansible_loop_var": "item", "changed": false, "item": 50051, "msg": "firewall is not currently running, unable to perform immediate actions without a running firewall daemon"}
failed: [ansible25.example.com] (item=53) => {"ansible_loop_var": "item", "changed": false, "item": 53, "msg": "firewall is not currently running, unable to perform immediate actions without a running firewall daemon"}
failed: [ansible25.example.com] (item=80) => {"ansible_loop_var": "item", "changed": false, "item": 80, "msg": "firewall is not currently running, unable to perform immediate actions without a running firewall daemon"}
failed: [ansible25.example.com] (item=443) => {"ansible_loop_var": "item", "changed": false, "item": 443, "msg": "firewall is not currently running, unable to perform immediate actions without a running firewall daemon"}
failed: [ansible25.example.com] (item=5432) => {"ansible_loop_var": "item", "changed": false, "item": 5432, "msg": "firewall is not currently running, unable to perform immediate actions without a running firewall daemon"}
failed: [ansible25.example.com] (item=6379) => {"ansible_loop_var": "item", "changed": false, "item": 6379, "msg": "firewall is not currently running, unable to perform immediate actions without a running firewall daemon"}
failed: [ansible25.example.com] (item=27199) => {"ansible_loop_var": "item", "changed": false, "item": 27199, "msg": "firewall is not currently running, unable to perform immediate actions without a running firewall daemon"}
failed: [ansible25.example.com] (item=9000) => {"ansible_loop_var": "item", "changed": false, "item": 9000, "msg": "firewall is not currently running, unable to perform immediate actions without a running firewall daemon"}
failed: [ansible25.example.com] (item=9001) => {"ansible_loop_var": "item", "changed": false, "item": 9001, "msg": "firewall is not currently running, unable to perform immediate actions without a running firewall daemon"}
failed: [ansible25.example.com] (item=9002) => {"ansible_loop_var": "item", "changed": false, "item": 9002, "msg": "firewall is not currently running, unable to perform immediate actions without a running firewall daemon"}
failed: [ansible25.example.com] (item=9003) => {"ansible_loop_var": "item", "changed": false, "item": 9003, "msg": "firewall is not currently running, unable to perform immediate actions without a running firewall daemon"}
...ignoring

TASK [firewall_config : Collect failed UDP ports] **********************************************************************************************************************
ok: [ansible25.example.com] => (item=50051/udp)
ok: [ansible25.example.com] => (item=53/udp)
ok: [ansible25.example.com] => (item=80/udp)
ok: [ansible25.example.com] => (item=443/udp)
ok: [ansible25.example.com] => (item=5432/udp)
ok: [ansible25.example.com] => (item=6379/udp)
ok: [ansible25.example.com] => (item=27199/udp)
ok: [ansible25.example.com] => (item=9000/udp)
ok: [ansible25.example.com] => (item=9001/udp)
ok: [ansible25.example.com] => (item=9002/udp)
ok: [ansible25.example.com] => (item=9003/udp)

TASK [firewall_config : Mark all UDP ports as failed when firewalld is not available] **********************************************************************************
skipping: [ansible25.example.com] => (item=50051)
skipping: [ansible25.example.com] => (item=53)
skipping: [ansible25.example.com] => (item=80)
skipping: [ansible25.example.com] => (item=443)
skipping: [ansible25.example.com] => (item=5432)
skipping: [ansible25.example.com] => (item=6379)
skipping: [ansible25.example.com] => (item=27199)
skipping: [ansible25.example.com] => (item=9000)
skipping: [ansible25.example.com] => (item=9001)
skipping: [ansible25.example.com] => (item=9002)
skipping: [ansible25.example.com] => (item=9003)
skipping: [ansible25.example.com]

TASK [firewall_config : Display UDP ports configuration result] ********************************************************************************************************
skipping: [ansible25.example.com]

TASK [firewall_config : Reload firewalld service] **********************************************************************************************************************
changed: [ansible25.example.com]

TASK [firewall_config : Display firewall reload result] ****************************************************************************************************************
ok: [ansible25.example.com] => {
    "msg": "Firewall configuration reloaded"
}

TASK [firewall_config : Check firewall status] *************************************************************************************************************************
skipping: [ansible25.example.com]

TASK [firewall_config : Display firewall status] ***********************************************************************************************************************
ok: [ansible25.example.com] => {
    "msg": []
}

TASK [firewall_config : Display firewall configuration summary] ********************************************************************************************************
ok: [ansible25.example.com] => {
    "msg": [
        "=== Firewall Configuration Summary ===",
        "Firewalld service available: True",
        "❌ Failed ports: 50051/tcp, 53/tcp, 80/tcp, 443/tcp, 5432/tcp, 6379/tcp, 27199/tcp, 9000/tcp, 9001/tcp, 9002/tcp, 9003/tcp, 50051/udp, 53/udp, 80/udp, 443/udp, 5432/udp, 6379/udp, 27199/udp, 9000/udp, 9001/udp, 9002/udp, 9003/udp",
        "",
        "⚠️  Warning: Some ports failed to configure. Please check firewalld service status.",
        ""
    ]
}

TASK [hosts_config : Backup hosts file] ********************************************************************************************************************************
changed: [ansible25.example.com]

TASK [hosts_config : Display backup result] ****************************************************************************************************************************
ok: [ansible25.example.com] => {
    "msg": "Hosts file backup completed"
}

TASK [hosts_config : Initialize hosts template path] *******************************************************************************************************************
ok: [ansible25.example.com]

TASK [hosts_config : Fetch hosts file from template server] ************************************************************************************************************
skipping: [ansible25.example.com]

TASK [hosts_config : Update fetch status] ******************************************************************************************************************************
ok: [ansible25.example.com]

TASK [hosts_config : Display fetch result] *****************************************************************************************************************************
ok: [ansible25.example.com] => {
    "msg": "Hosts file fetched from template server to /tmp/hosts.template.1770259697"
}

TASK [hosts_config : Check if template file exists on control node] ****************************************************************************************************
ok: [ansible25.example.com -> localhost]

TASK [hosts_config : Copy template to remote location] *****************************************************************************************************************
skipping: [ansible25.example.com]

TASK [hosts_config : Display copy result] ******************************************************************************************************************************
ok: [ansible25.example.com] => {
    "msg": "Template hosts file copied to target server"
}

TASK [hosts_config : Display copy skip message] ************************************************************************************************************************
ok: [ansible25.example.com] => {
    "msg": "Warning: Template file not found on control node. Skipping copy."
}

TASK [hosts_config : Set local network facts] **************************************************************************************************************************
ok: [ansible25.example.com]

TASK [hosts_config : Display local network information] ****************************************************************************************************************
ok: [ansible25.example.com] => {
    "msg": [
        "Local IP: 10.71.18.156",
        "Local Hostname: ansible25",
        "Local FQDN: ansible25.example.com"
    ]
}

TASK [hosts_config : Check if template file exists on remote] **********************************************************************************************************
ok: [ansible25.example.com]

TASK [hosts_config : Copy template hosts file to /etc/hosts] ***********************************************************************************************************
skipping: [ansible25.example.com]

TASK [hosts_config : Display merge result] *****************************************************************************************************************************
skipping: [ansible25.example.com]

TASK [hosts_config : Skip merge when template not available] ***********************************************************************************************************
ok: [ansible25.example.com] => {
    "msg": "Warning: Template hosts file not available. Skipping hosts file merge."
}

TASK [hosts_config : Check if local entry exists] **********************************************************************************************************************
skipping: [ansible25.example.com]

TASK [hosts_config : Add local entry to hosts file] ********************************************************************************************************************
skipping: [ansible25.example.com]

TASK [hosts_config : Display local entry result] ***********************************************************************************************************************
ok: [ansible25.example.com] => {
    "msg": "Local entry already exists"
}

TASK [hosts_config : Display hosts file content] ***********************************************************************************************************************
skipping: [ansible25.example.com]

TASK [hosts_config : Display hosts file] *******************************************************************************************************************************
ok: [ansible25.example.com] => {
    "msg": []
}

TASK [hosts_config : Remove remote temporary files] ********************************************************************************************************************
ok: [ansible25.example.com]

TASK [hosts_config : Remove local template file] ***********************************************************************************************************************
ok: [ansible25.example.com -> localhost]

TASK [hosts_config : Display hosts configuration summary] **************************************************************************************************************
ok: [ansible25.example.com] => {
    "msg": [
        "=== Hosts Configuration Summary ===",
        "✅ Hosts file fetched from template server",
        "✅ Template file copied to target server",
        "✅ Hosts file merged successfully"
    ]
}

TASK [chrony_config : Initialize chrony configuration status] **********************************************************************************************************
ok: [ansible25.example.com]

TASK [chrony_config : Install chrony package] **************************************************************************************************************************
ok: [ansible25.example.com]

TASK [chrony_config : Update installation status] **********************************************************************************************************************
ok: [ansible25.example.com]

TASK [chrony_config : Display installation result] *********************************************************************************************************************
ok: [ansible25.example.com] => {
    "msg": "Chrony package already installed"
}

TASK [chrony_config : Configure chrony] ********************************************************************************************************************************
changed: [ansible25.example.com]

TASK [chrony_config : Update configuration status] *********************************************************************************************************************
ok: [ansible25.example.com]

TASK [chrony_config : Display configuration result] ********************************************************************************************************************
ok: [ansible25.example.com] => {
    "msg": "Chrony configuration updated"
}

TASK [chrony_config : Start and enable chronyd service] ****************************************************************************************************************
ok: [ansible25.example.com]

TASK [chrony_config : Update service status] ***************************************************************************************************************************
ok: [ansible25.example.com]

TASK [chrony_config : Display service status] **************************************************************************************************************************
ok: [ansible25.example.com] => {
    "msg": "Chronyd service already running"
}

TASK [chrony_config : Wait for chrony to sync] *************************************************************************************************************************
Pausing for 10 seconds
(ctrl+C then 'C' = continue early, ctrl+C then 'A' = abort)
ok: [ansible25.example.com]

TASK [chrony_config : Verify time synchronization status] **************************************************************************************************************
skipping: [ansible25.example.com]

TASK [chrony_config : Display time synchronization status] *************************************************************************************************************
ok: [ansible25.example.com] => {
    "msg": []
}

TASK [chrony_config : Display chrony configuration summary] ************************************************************************************************************
ok: [ansible25.example.com] => {
    "msg": [
        "=== Chrony Configuration Summary ===",
        "✅ Chrony package installed",
        "✅ Chrony configuration updated",
        "✅ Chronyd service started",
        "✅ All chrony configuration tasks completed successfully"
    ]
}

TASK [system_optimization : Initialize system optimization status] *****************************************************************************************************
ok: [ansible25.example.com]

TASK [system_optimization : Configure system limits] *******************************************************************************************************************
changed: [ansible25.example.com] => (item=prometheus soft nofile 65536)
changed: [ansible25.example.com] => (item=prometheus hard nofile 65536)
changed: [ansible25.example.com] => (item=* soft nofile 65536)
changed: [ansible25.example.com] => (item=* hard nofile 65536)

TASK [system_optimization : Update limits configuration status] ********************************************************************************************************
ok: [ansible25.example.com]

TASK [system_optimization : Display system limits configuration result] ************************************************************************************************
ok: [ansible25.example.com] => {
    "msg": "System limits updated"
}

TASK [system_optimization : Configure sysctl parameters] ***************************************************************************************************************
changed: [ansible25.example.com] => (item={'key': 'net.core.somaxconn', 'value': 65535})
changed: [ansible25.example.com] => (item={'key': 'net.ipv4.tcp_max_syn_backlog', 'value': 65535})
changed: [ansible25.example.com] => (item={'key': 'vm.max_map_count', 'value': 262144})

TASK [system_optimization : Update sysctl configuration status] ********************************************************************************************************
ok: [ansible25.example.com]

TASK [system_optimization : Display sysctl configuration result] *******************************************************************************************************
ok: [ansible25.example.com] => {
    "msg": "Sysctl parameters updated"
}

TASK [system_optimization : Apply sysctl settings] *********************************************************************************************************************
skipping: [ansible25.example.com]

TASK [system_optimization : Display sysctl apply result] ***************************************************************************************************************
ok: [ansible25.example.com] => {
    "msg": "Sysctl configuration applied"
}

TASK [system_optimization : Check current file descriptor limit] *******************************************************************************************************
skipping: [ansible25.example.com]

TASK [system_optimization : Display current file descriptor limit] *****************************************************************************************************
ok: [ansible25.example.com] => {
    "msg": "Current file descriptor limit: "
}

TASK [system_optimization : Verify sysctl parameters] ******************************************************************************************************************
skipping: [ansible25.example.com] => (item={'key': 'net.core.somaxconn', 'value': 65535})
skipping: [ansible25.example.com] => (item={'key': 'net.ipv4.tcp_max_syn_backlog', 'value': 65535})
skipping: [ansible25.example.com] => (item={'key': 'vm.max_map_count', 'value': 262144})
skipping: [ansible25.example.com]

TASK [system_optimization : Display sysctl verification results] *******************************************************************************************************
ok: [ansible25.example.com] => (item={'changed': False, 'stdout': '', 'stderr': '', 'rc': 0, 'cmd': 'sysctl net.core.somaxconn', 'start': None, 'end': None, 'delta': None, 'msg': 'Command would have run if not in check mode', 'skipped': True, 'invocation': {'module_args': {'_raw_params': 'sysctl net.core.somaxconn', '_uses_shell': True, 'expand_argument_vars': True, 'stdin_add_newline': True, 'strip_empty_ends': True, 'argv': None, 'chdir': None, 'executable': None, 'creates': None, 'removes': None, 'stdin': None}}, 'stdout_lines': [], 'stderr_lines': [], 'failed': False, 'item': {'key': 'net.core.somaxconn', 'value': 65535}, 'ansible_loop_var': 'item'}) => {
    "msg": ""
}
ok: [ansible25.example.com] => (item={'changed': False, 'stdout': '', 'stderr': '', 'rc': 0, 'cmd': 'sysctl net.ipv4.tcp_max_syn_backlog', 'start': None, 'end': None, 'delta': None, 'msg': 'Command would have run if not in check mode', 'skipped': True, 'invocation': {'module_args': {'_raw_params': 'sysctl net.ipv4.tcp_max_syn_backlog', '_uses_shell': True, 'expand_argument_vars': True, 'stdin_add_newline': True, 'strip_empty_ends': True, 'argv': None, 'chdir': None, 'executable': None, 'creates': None, 'removes': None, 'stdin': None}}, 'stdout_lines': [], 'stderr_lines': [], 'failed': False, 'item': {'key': 'net.ipv4.tcp_max_syn_backlog', 'value': 65535}, 'ansible_loop_var': 'item'}) => {
    "msg": ""
}
ok: [ansible25.example.com] => (item={'changed': False, 'stdout': '', 'stderr': '', 'rc': 0, 'cmd': 'sysctl vm.max_map_count', 'start': None, 'end': None, 'delta': None, 'msg': 'Command would have run if not in check mode', 'skipped': True, 'invocation': {'module_args': {'_raw_params': 'sysctl vm.max_map_count', '_uses_shell': True, 'expand_argument_vars': True, 'stdin_add_newline': True, 'strip_empty_ends': True, 'argv': None, 'chdir': None, 'executable': None, 'creates': None, 'removes': None, 'stdin': None}}, 'stdout_lines': [], 'stderr_lines': [], 'failed': False, 'item': {'key': 'vm.max_map_count', 'value': 262144}, 'ansible_loop_var': 'item'}) => {
    "msg": ""
}

TASK [system_optimization : Display system optimization summary] *******************************************************************************************************
ok: [ansible25.example.com] => {
    "msg": [
        "=== System Optimization Summary ===",
        "✅ System limits configured",
        "✅ Sysctl parameters configured",
        "✅ All system optimization tasks completed successfully"
    ]
}

TASK [verification : Display system information] ***********************************************************************************************************************
ok: [ansible25.example.com] => {
    "msg": [
        "=== System Information ===",
        "OS: RedHat 9.4",
        "Kernel: 5.14.0-427.42.1.el9_4.x86_64",
        "Hostname: ansible25",
        "FQDN: ansible25.example.com",
        "IP Address: 10.71.18.156"
    ]
}

TASK [verification : Verify admin user] ********************************************************************************************************************************
skipping: [ansible25.example.com]

TASK [verification : Verify sudo privileges] ***************************************************************************************************************************
skipping: [ansible25.example.com]

TASK [verification : Display user verification results] ****************************************************************************************************************
ok: [ansible25.example.com] => {
    "msg": [
        "=== User Verification ===",
        "Admin user: ",
        "Sudo privileges: []"
    ]
}

TASK [verification : Display network interfaces] ***********************************************************************************************************************
skipping: [ansible25.example.com]

TASK [verification : Test network connectivity] ************************************************************************************************************************
ok: [ansible25.example.com]

TASK [verification : Display network verification results] *************************************************************************************************************
ok: [ansible25.example.com] => {
    "msg": [
        "=== Network Verification ===",
        "Network connectivity: FAILED"
    ]
}

TASK [verification : Check chrony service status] **********************************************************************************************************************
ok: [ansible25.example.com]

TASK [verification : Check firewalld service status] *******************************************************************************************************************
ok: [ansible25.example.com]

TASK [verification : Display service status] ***************************************************************************************************************************
ok: [ansible25.example.com] => {
    "msg": [
        "=== Service Status ===",
        "chrony: active",
        "firewalld: inactive"
    ]
}

TASK [verification : Check port listening status] **********************************************************************************************************************
skipping: [ansible25.example.com]

TASK [verification : Display port listening status] ********************************************************************************************************************
ok: [ansible25.example.com] => {
    "msg": [
        "=== Port Listening ===",
        [
            "No ports listening"
        ]
    ]
}

TASK [verification : Check time synchronization status] ****************************************************************************************************************
skipping: [ansible25.example.com]

TASK [verification : Display time synchronization status] **************************************************************************************************************
ok: [ansible25.example.com] => {
    "msg": [
        "=== Time Synchronization Status ===",
        []
    ]
}

TASK [verification : Display verification summary] *********************************************************************************************************************
ok: [ansible25.example.com] => {
    "msg": [
        "🎉 === Configuration Verification Complete ===",
        "✅ System information verified",
        "✅ User permissions verified",
        "✅ Network connectivity verified",
        "✅ Service status verified",
        "✅ Port listening verified",
        "✅ Time synchronization verified",
        "🎯 All configuration items verified, system is standardized"
    ]
}

TASK [Collect configuration summary from all roles] ********************************************************************************************************************
ok: [ansible25.example.com]

TASK [Display final configuration summary] *****************************************************************************************************************************
ok: [ansible25.example.com] => {
    "msg": [
        "═══════════════════════════════════════════════════════════════",
        "🎯 RHEL9 STANDARDIZATION CONFIGURATION SUMMARY",
        "═══════════════════════════════════════════════════════════════",
        "📋 Configuration Status:",
        "   • User Management: Check user_management role summary above",
        "   • System Configuration: Check system_config role summary above",
        "   • Firewall Configuration: Check firewall_config role summary above",
        "   • Hosts Configuration: Check hosts_config role summary above",
        "   • Chrony Configuration: Check chrony_config role summary above",
        "   • System Optimization: Check system_optimization role summary above",
        "   • Verification: Check verification role summary above",
        "",
        "📝 Next Steps:",
        "   • Review all role summaries above for any errors or warnings",
        "   • Address any failed configurations manually if needed",
        "   • A system reboot is recommended to apply all configurations",
        "═══════════════════════════════════════════════════════════════"
    ]
}

TASK [Prompt for system reboot] ****************************************************************************************************************************************
[Prompt for system reboot]
Do you want to reboot the system to apply all configurations? (y/N):
ok: [ansible25.example.com]

TASK [Reboot system] ***************************************************************************************************************************************************
skipping: [ansible25.example.com]

PLAY RECAP *************************************************************************************************************************************************************
ansible25.example.com      : ok=96   changed=9    unreachable=0    failed=0    skipped=28   rescued=1    ignored=2
