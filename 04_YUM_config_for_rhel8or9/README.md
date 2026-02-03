# Automated YUM Repository Deployment

## Overview

This playbook automates the deployment of an internal YUM/DNF repository server and configures all client hosts to use it. It supports deployment from RHEL ISO images and can use HTTP, HTTPS, or FTP protocols for repository access. This solution is ideal for offline/air-gapped environments where internet access is not available.

## Features

- **OS Compatibility**: Supports RHEL 8/9 only (RHEL 7 is not supported)
- **Multiple Protocol Support**: HTTP, HTTPS, and FTP
- **ISO-based Deployment**: Automatically mounts and extracts content from RHEL ISO images
- **Automatic Configuration**:
  - SELinux contexts for web/FTP services
  - Firewall rules (firewalld) for HTTP/HTTPS/FTP ports
  - HTTPD ServerName configuration to prevent HTTP 400 errors
  - Network access SELinux booleans
- **Dynamic Server Discovery**: Automatically uses server IP address from Ansible facts (no hardcoded IPs)
- **Modular Design**: Separate tasks for server and client configuration
- **Comprehensive Validation**: OS compatibility, parameter validation, and error handling
- **Production Ready**: Tested and verified in real environments

## Files

- `yum_repo_deployment.yml` - Main playbook file
- `yum_repo_server_tasks.yml` - Server deployment tasks
- `yum_repo_client_tasks.yml` - Client configuration tasks
- `README.md` - This file

## Prerequisites

- Ansible 2.9 or later
- Ansible control node with network access to target hosts
- Target hosts: RHEL 8/9 only (RHEL 7 is not supported)
- SSH access to target hosts with root privileges (or sudo access)
- RHEL ISO image file (e.g., `rhel-8.10-x86_64-dvd.iso`)
- `ansible.posix` collection installed (for mount and synchronize modules)

### Installing Required Collection

Before running this playbook, ensure the `ansible.posix` collection is installed:

```bash
ansible-galaxy collection install ansible.posix -p collections/ --force
```

## Usage

### 1. Prepare RHEL ISO Image

Upload your RHEL ISO image file to the repository server. For example:

```bash
# On the repository server
mkdir -p /data
# Upload rhel-8.10-x86_64-dvd.iso to /data/
```

### 2. Update Playbook Variables

Edit `yum_repo_deployment.yml` to configure your environment:

**For Server Play:**

```yaml
vars:
  protocol: "http"                    # http, https, or ftp
  source: "iso"                       # Source type
  documentRoot: "/var/www/html"       # Web server document root
  isoPath: "/data/rhel-8.10-x86_64-dvd.iso"  # Path to ISO file
  relativePath: "RHEL-8.10/RedHatEnterpriseLinux/x86_64"
  mountPath: "/mnt/RHEL-8.10/RedHatEnterpriseLinux/x86_64"
```

**For Client Play:**

```yaml
vars:
  protocol: "http"                    # Must match server protocol
  release: "8.10"                     # RHEL release version
  relativePath: "RHEL-8.10/RedHatEnterpriseLinux/x86_64"
  repoName: "remote_repo_{{ release }}-{{ protocol }}"
```

### 3. Prepare Inventory File

Create an inventory file (e.g., `inventory`) with server and client groups:

```ini
[server]
# Repository server hostname or IP
192.168.100.1

[client]
# Client hosts that will use the repository (RHEL 8/9 only)
TEST-RHEL-8.9-1
ansible25.example.com
```

**Important Notes**:

- This playbook only supports RHEL 8/9. RHEL 7 hosts should not be included in the client group.
- The playbook automatically discovers the repository server IP address from Ansible facts using `hostvars[groups['server'][0]]['ansible_default_ipv4']['address']`, ensuring clients use the IP address instead of hostname to avoid DNS resolution issues.
- You can use either hostname or IP address in the inventory for the server group - the playbook will automatically extract the IP address.

### 4. Install Required Packages on Server

Ensure the appropriate service package is installed on the repository server:

**For HTTP/HTTPS:**

```bash
yum install -y httpd
```

**For FTP:**

```bash
yum install -y vsftpd
```

**Note**: The playbook automatically:

- Configures firewall rules (firewalld) to allow HTTP/HTTPS/FTP traffic
- Configures HTTPD ServerName to prevent HTTP 400 errors
- Sets SELinux booleans for network access (e.g., `httpd_can_network_connect`)
- Ensures `firewalld` service is running and enabled

Ensure `firewalld` is installed on the repository server (it's typically installed by default on RHEL 8/9).

### 5. Run the Playbook

Execute the playbook from the control node:

```bash
ansible-playbook yum_repo_deployment.yml -i inventory
```

### 6. Verify Deployment

After execution, verify the repository is working:

**On the server:**

```bash
# Check if files are in place
ls -la /var/www/html/RHEL-8.10/RedHatEnterpriseLinux/x86_64/

# Check if web service is running
systemctl status httpd
```

**On a client:**

```bash
# List configured repositories
yum repolist enabled
# Expected output: Should show remote-repo-baseos-* and remote-repo-appstream-* repositories

# Test repository access and search for packages
yum search vim
# Expected output: Should list vim packages from the remote repository

# Verify repository configuration file
cat /etc/yum.repos.d/remote_repo_*.repo
# Should show baseurl pointing to server IP address (not hostname)
```

**Expected Successful Output:**

```
repo id                              repo name
remote-repo-appstream-8.10           Remote Repository AppStream 8.10
remote-repo-baseos-8.10              Remote Repository BaseOS 8.10
```

## Customization

### Change Protocol

Modify the `protocol` variable in both server and client plays:

```yaml
vars:
  protocol: "https"  # or "ftp"
```

**Note**: For HTTPS, you'll need to configure SSL certificates. For FTP, ensure `vsftpd` is installed and configured.

### Change Repository Paths

Modify the `relativePath` and `mountPath` variables to match your ISO structure:

```yaml
vars:
  relativePath: "RHEL-9.0/RedHatEnterpriseLinux/x86_64"
  mountPath: "/mnt/RHEL-9.0/RedHatEnterpriseLinux/x86_64"
```

### Change Document Root

Modify the `documentRoot` variable for the server:

```yaml
vars:
  documentRoot: "/var/www/html"
```

## Playbook Structure

### Main Playbook (`yum_repo_deployment.yml`)

1. **Server Play**: Sets up the repository server

   - OS compatibility validation
   - Parameter validation
   - Includes server tasks
2. **Client Play**: Configures client hosts

   - OS compatibility validation
   - Parameter validation
   - Includes client tasks

### Server Tasks (`yum_repo_server_tasks.yml`)

1. **Pre-check**: Verify required packages (httpd/vsftpd)
2. **Setup**: Create directories (document root, mount point, repository path)
3. **Mount**: Mount ISO image read-only
4. **Synchronize**: Copy files from ISO to document root using `ansible.posix.synchronize`
5. **Unmount**: Unmount ISO after synchronization
6. **Service Configuration**:
   - Configure HTTPD ServerName
   - Restart web/FTP service
7. **Firewall**: Configure firewalld rules for HTTP/HTTPS/FTP
8. **SELinux**:
   - Apply proper file contexts (`httpd_sys_content_t` or `public_content_t`)
   - Set network access booleans
9. **Verification**: Verify repository metadata (repodata, repomd.xml) exists

### Client Tasks (`yum_repo_client_tasks.yml`)

1. **Repository Configuration**: Create YUM repository files pointing to server IP address
   - BaseOS repository: Core OS packages
   - AppStream repository: Application packages
2. **Cache Management**: Clean YUM cache to ensure fresh metadata
3. **Verification**: Verify repository accessibility using `yum repolist enabled`

## Protocol Support

### HTTP/HTTPS

- Requires `httpd` package
- Uses `/var/www/html` as document root (default)
- Applies `httpd_sys_content_t` SELinux context

### FTP

- Requires `vsftpd` package
- Uses `/var/ftp/pub` as document root (typically)
- Applies `public_content_t` SELinux context

## Best Practices

- **Test First**: Run in check mode first: `ansible-playbook yum_repo_deployment.yml -i inventory --check`
- **Use Tags**: Use tags for selective execution:
  - `--tags validation` - Run only validation tasks
  - `--tags server` - Run only server tasks
  - `--tags client` - Run only client tasks
  - `--tags mount` - Run only mount tasks
  - `--tags sync` - Run only synchronization tasks
- **Maintenance Windows**: Schedule repository updates during maintenance windows
- **Backup**: Keep backups of ISO images and repository configurations
- **Monitoring**: Monitor disk space on repository server
- **Verify After Deployment**: Always verify repository access from clients using `yum repolist` and `yum search <package>`

## Post-Deployment Verification

After successful playbook execution, perform the following verification steps:

### 1. Server Verification

```bash
# Verify repository files are in place
ls -la /var/www/html/RHEL-8.10/RedHatEnterpriseLinux/x86_64/BaseOS/repodata/
ls -la /var/www/html/RHEL-8.10/RedHatEnterpriseLinux/x86_64/AppStream/repodata/

# Verify httpd is running and listening
systemctl status httpd
ss -tlnp | grep ':80'
# Should show: LISTEN 0 128 0.0.0.0:80

# Verify firewall rules
firewall-cmd --list-all
# Should show http service or port 80/tcp

# Test local access
curl http://localhost/RHEL-8.10/RedHatEnterpriseLinux/x86_64/BaseOS/repodata/repomd.xml
# Should return XML content
```

### 2. Client Verification

```bash
# Verify repository configuration
cat /etc/yum.repos.d/remote_repo_*.repo
# Check that baseurl uses IP address (not hostname)

# List repositories
yum repolist enabled
# Should show remote-repo-baseos-* and remote-repo-appstream-* repositories

# Test package search
yum search vim
# Should list packages from remote repository

# Test package installation (optional)
yum install -y vim-minimal --downloadonly
# Should download from remote repository
```

### 3. Expected Results

- **Server**: Repository files accessible via HTTP, httpd running, firewall configured
- **Client**: Repositories listed in `yum repolist`, packages searchable, metadata downloads successfully
- **Network**: Clients can access repository using server IP address

## Troubleshooting

### Playbook fails with "Unsupported operating system"

- Verify target host OS is RHEL 8/9 (RHEL 7 is not supported)
- Check `ansible_distribution` and `ansible_distribution_major_version` facts
- Remove any RHEL 7 hosts from the inventory

### "httpd/vsftpd package is not installed"

- Install the required package on the repository server:
  ```bash
  yum install -y httpd    # For HTTP/HTTPS
  yum install -y vsftpd   # For FTP
  ```

### "ISO file does not exist"

- Verify the ISO file path is correct
- Ensure the ISO file is uploaded to the repository server
- Check file permissions (must be readable)

### "Failed to synchronize repository files"

- Check disk space on repository server
- Verify ISO is properly mounted
- Check SELinux status and permissions
- Review playbook output for specific error messages

### "Repository not accessible from clients"

- **Verify web/FTP service is running on server:**

  ```bash
  systemctl status httpd
  ss -tlnp | grep ':80'
  ```
- **Check firewall rules:**

  ```bash
  firewall-cmd --list-all
  # Should show ports 80/tcp (HTTP) or 443/tcp (HTTPS) or ftp service
  ```

  The playbook automatically configures firewall rules, but you can manually add them:

  ```bash
  firewall-cmd --permanent --add-service=http
  firewall-cmd --reload
  ```
- **Verify SELinux contexts are correctly applied:**

  ```bash
  ls -Z /var/www/html/RHEL-8.10/RedHatEnterpriseLinux/x86_64/
  # Should show httpd_sys_content_t context
  ```
- **Test repository URL manually from client:**

  ```bash
  curl http://server-ip/RHEL-8.10/RedHatEnterpriseLinux/x86_64/repodata/repomd.xml
  ```
- **Check if httpd is listening on all interfaces (not just localhost):**

  ```bash
  ss -tlnp | grep ':80'
  # Should show 0.0.0.0:80 or *:80, not 127.0.0.1:80
  ```
- **Verify network connectivity:**

  ```bash
  # From client, test connectivity to server
  ping server-ip
  telnet server-ip 80
  ```

### "ansible.posix collection not found"

- Install the collection:
  ```bash
  ansible-galaxy collection install ansible.posix -p collections/ --force
  ```

### Client cannot access repository

- **Verify network connectivity between client and server:**

  ```bash
  ping server-ip
  telnet server-ip 80
  ```
- **Check repository configuration file:**

  ```bash
  cat /etc/yum.repos.d/remote_repo_*.repo
  # Verify baseurl uses IP address (not hostname)
  # Verify paths match server configuration
  ```
- **Test repository URL from client:**

  ```bash
  curl http://server-ip/RHEL-8.10/RedHatEnterpriseLinux/x86_64/BaseOS/repodata/repomd.xml
  # Should return XML content
  ```
- **Clean YUM cache and retry:**

  ```bash
  yum clean all
  yum repolist
  ```
- **Check if firewall is blocking access:**

  ```bash
  # On server
  firewall-cmd --list-all
  # Should show http service or port 80/tcp
  ```
- **Verify SELinux is not blocking:**

  ```bash
  # On server
  getsebool httpd_can_network_connect
  # Should show "on"
  ```

### HTTP 400 Bad Request error

- **Symptom**: Client receives "400 Bad Request" when accessing repository URL
- **Cause**: httpd ServerName not configured, causing httpd to reject requests
- **Solution**: The playbook automatically configures ServerName during execution. If issue persists after running the playbook:
  ```bash
  # On server, check httpd configuration
  grep ServerName /etc/httpd/conf/httpd.conf

  # If missing, add ServerName (playbook does this automatically)
  echo "ServerName $(hostname):80" >> /etc/httpd/conf/httpd.conf
  systemctl restart httpd
  ```
- **Verify**: Test from client:
  ```bash
  curl http://server-ip/RHEL-8.10/RedHatEnterpriseLinux/x86_64/BaseOS/repodata/repomd.xml
  # Should return XML content, not HTML error page
  ```

### "Couldn't resolve host name" error

- **Symptom**: Client cannot resolve server hostname
- **Cause**: DNS resolution failure or client repository config using hostname instead of IP
- **Solution**: The playbook automatically uses server IP address from Ansible facts. Verify the repository configuration:
  ```bash
  # On client, check repository configuration
  cat /etc/yum.repos.d/remote_repo_*.repo
  # baseurl should use IP address, not hostname

  # If it shows hostname, re-run the playbook to update configuration
  ```
- **Note**: The playbook uses `hostvars[groups['server'][0]]['ansible_default_ipv4']['address']` to ensure IP addresses are used, avoiding DNS issues.

## Security Considerations

- **Root Access Required**: This playbook requires root privileges
- **SELinux**: Proper SELinux contexts are automatically configured
- **Firewall**: Ensure firewall rules allow access to repository services
- **Network Security**: Use HTTPS in production environments when possible
- **Access Control**: Configure web server access controls if needed

## Example Inventory

```ini
[server]
# Repository server - can use hostname or IP address
# The playbook will automatically extract IP address from Ansible facts
TEST-RHEL-8.9-1

[client]
# Client hosts that will use the repository (RHEL 8/9 only)
# Can use hostname or IP address
10.66.208.248
ansible25.example.com
```

**Note**:

- Only include RHEL 8/9 hosts in the `[client]` group
- The playbook automatically uses the server's IP address (from Ansible facts) in client repository configurations, avoiding DNS resolution issues
- You can use either hostname or IP address in the inventory - both work correctly

## Example Output

### Server Setup

```
PLAY [Setup the remote yum repo server] ******************************

TASK [Validate OS compatibility (RHEL 8/9 only)] **********************
ok: [TEST-RHEL-8.9-1] => {
    "msg": "OS compatibility check passed: RedHat 8"
}

TASK [Mount ISO read-only] *********************************************
changed: [TEST-RHEL-8.9-1]

TASK [Copy repo files] *************************************************
changed: [TEST-RHEL-8.9-1]

TASK [Configure httpd ServerName] *************************************
changed: [TEST-RHEL-8.9-1]

TASK [Restart service httpd, in all cases] *****************************
changed: [TEST-RHEL-8.9-1]

TASK [Apply httpd SELinux file context to filesystem] ******************
changed: [TEST-RHEL-8.9-1]

PLAY RECAP *************************************************************
TEST-RHEL-8.9-1            : ok=26   changed=7    failed=0
```

### Client Configuration

```
PLAY [Setup the remote yum repo client] ******************************

TASK [Validate OS compatibility (RHEL 8/9 only)] **********************
ok: [10.66.208.248] => {
    "msg": "OS compatibility check passed: RedHat 8"
}

TASK [Add repositories for release >= RHEL-8.0] ***********************
changed: [10.66.208.248] => (item={'name': 'remote-repo-baseos-8.10', ...})
changed: [10.66.208.248] => (item={'name': 'remote-repo-appstream-8.10', ...})

TASK [Clean YUM cache] *************************************************
changed: [10.66.208.248]

TASK [Verify repository accessibility] ********************************
ok: [10.66.208.248]

TASK [Display repository list for debugging] **************************
ok: [10.66.208.248] => {
    "msg": [
        "repo id                              repo name",
        "remote-repo-appstream-8.10           Remote Repository AppStream 8.10",
        "remote-repo-baseos-8.10              Remote Repository BaseOS 8.10"
    ]
}

PLAY RECAP *************************************************************
10.66.208.248              : ok=11   changed=2    failed=0
```

### Successful Client Verification

After deployment, verify the repository works correctly:

```bash
# List repositories
[root@client ~]# yum repolist
repo id                              repo name
remote-repo-appstream-8.10           Remote Repository AppStream 8.10
remote-repo-baseos-8.10              Remote Repository BaseOS 8.10

# Search for packages
[root@client ~]# yum search vim
Remote Repository BaseOS 8.10         | 2.4 MB     00:01
Remote Repository AppStream 8.10      | 7.5 MB     00:01
===================================================================== Name & Summary Matched: vim =====================================================================
vim-X11.x86_64 : The VIM version of the vi editor for the X Window System
vim-common.x86_64 : The common files needed by any version of the VIM editor
vim-enhanced.x86_64 : A version of the VIM editor which includes recent enhancements
...
```

## Author

GCG AAP SSA Team + v3.01 20260217

## License

📜 License Type: End User License Agreement (EULA)
🔒 Authorization: Subscription-based License
