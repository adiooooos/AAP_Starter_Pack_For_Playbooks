# MariaDB Deployment and Configuration Automation

## Overview

This Ansible Role-based solution provides automated MariaDB server deployment and configuration across multiple Linux hosts. It enables:

- **MariaDB Server Installation**: Automated package installation and service management
- **Security Hardening**: Root password configuration, anonymous user removal, test database removal
- **User and Database Management**: Create databases and users with proper permissions
- **Initial Backup**: Automatic database backup after deployment
- **Firewall Configuration**: Automatic firewalld configuration for MySQL service

The implementation follows AAP Starter Pack guidelines with FQDN modules, OS compatibility assertions, robust error handling, and clear debugging support.

## Features

- **Role-Based Architecture**: Modular, reusable, and maintainable design
- **OS Compatibility**: Supports RHEL 7/8/9, CentOS 7/8/9, Rocky Linux 8/9, AlmaLinux 8/9, and Oracle Linux 7/8/9
- **Security First**: Automatic security hardening (root password, remove anonymous users, remove test database)
- **Comprehensive Validation**: OS compatibility checks and parameter validation
- **Error Handling**: Robust error handling with `block-rescue` for critical changes
- **Firewall Management**: Automatic firewalld configuration for MySQL service
- **Password Protection**: Uses `no_log: true` to prevent password exposure in logs
- **Initial Backup**: Automatic database backup after deployment

## Files

- `deploy_mariadb_site.yml` - Main playbook for MariaDB server deployment
- `install_collection.sh` - Installation script for community.mysql collection
- `requirements.yml` - Collection requirements file
- `roles/mariadb_server/` - MariaDB server role
  - `defaults/main.yml` - Default variables (passwords, user names, database names)
  - `tasks/main.yml` - Core role tasks
- `README.md` - This file

## Prerequisites

- Ansible 2.9 or later
- Ansible control node with network access to target hosts
- Target hosts: RHEL 7/8/9, CentOS 7/8/9, Rocky Linux 8/9, AlmaLinux 8/9, or Oracle Linux 7/8/9
- SSH access to target hosts with root privileges (or sudo access)
- **Python MySQL library**: `python3-PyMySQL` must be installed on target hosts (automatically installed by playbook)
- **community.mysql collection**: Must be installed on control node (see Installation section below)
- firewalld service (optional, can be disabled via variables)

## Installation

### Install community.mysql Collection

Before running the MariaDB playbook, you must install the `community.mysql` collection on your Ansible control node.

#### Method 1: Using Installation Script (Recommended for AAP)

This project includes an installation script that installs the collection in the local `collections/` directory, which is ideal for AAP environments:

```bash
chmod +x install_collection.sh
./install_collection.sh
```

This script will:
- Check if `ansible-galaxy` is available
- Create a `collections/` directory in the project root
- Install `community.mysql` collection to the local directory
- Verify the installation

**Advantages for AAP users:**
- Collection is installed in the project directory
- No need for global collection installation
- Works seamlessly with AAP project sync

#### Method 2: Using ansible-galaxy (Global Installation)

```bash
ansible-galaxy collection install community.mysql
```

#### Method 3: Install from requirements.yml

This project includes a `requirements.yml` file. Install using:

```bash
ansible-galaxy collection install -r requirements.yml
```

Or install to local directory:

```bash
ansible-galaxy collection install -r requirements.yml -p ./collections
```

#### Verify Installation

After installation, verify the collection is available:

```bash
# For local installation
ansible-galaxy collection list -p ./collections | grep community.mysql

# For global installation
ansible-galaxy collection list | grep community.mysql
```

You should see output like:

```
community.mysql    <version>
```

## Usage

### 1. Prepare Inventory File

Create an inventory file (e.g., `inventory`) with your target hosts:

```ini
[db_servers]
servera.example.com
```

**Important Notes**:

- This playbook supports RHEL 7/8/9 and CentOS 7/8/9
- You can organize hosts into groups as needed
- The playbook will validate OS compatibility before execution

### 2. Customize Configuration

Edit `roles/mariadb_server/defaults/main.yml` to customize:

```yaml
# Root password (MUST be changed in production, use Ansible Vault!)
mariadb_root_password: "YourSecureRootPassword"

# General user configuration
mariadb_general_user_name: "webapp_user"
mariadb_general_user_password: "AnotherSecurePassword"

# Database configuration
mariadb_database_name: "webapp_db"
```

**Security Warning**: In production environments, use Ansible Vault to encrypt passwords! Do not store plaintext passwords in the defaults file.

### 3. Run the Playbook

```bash
ansible-playbook -i inventory deploy_mariadb_site.yml
```

This will:
- Install MariaDB server and python3-PyMySQL packages
- Start and enable MariaDB service
- Set root password for all root@host variants
- Create /root/.my.cnf for passwordless login
- Remove anonymous users
- Remove test database
- Create specified database
- Create general user with password
- Grant database privileges to general user
- Create initial database backup
- Configure firewalld to allow MySQL service

## Playbook Structure

### Main Playbook (`deploy_mariadb_site.yml`)

- **OS Compatibility Validation**: Checks for RHEL/CentOS 7/8/9
- **Role Execution**: Executes `mariadb_server` role on all target hosts

### Role Structure (`roles/mariadb_server/`)

- **`defaults/main.yml`**: Default variables (passwords, user names, database names)
- **`tasks/main.yml`**: Core role tasks organized into sections:
  1. Pre-flight Checks
  2. Package Installation
  3. Service Management
  4. Security Hardening
  5. Database and User Configuration
  6. Initial Backup
  7. Firewall Configuration

## Customization Examples

### Example 1: Customize Database and User Names

Edit `roles/mariadb_server/defaults/main.yml`:

```yaml
mariadb_general_user_name: "myapp_user"
mariadb_database_name: "myapp_db"
```

### Example 2: Customize Backup Location

Edit `roles/mariadb_server/defaults/main.yml`:

```yaml
mariadb_backup_path: "/var/backups/mariadb"
mariadb_backup_filename: "{{ mariadb_database_name }}_backup_{{ ansible_date_time.epoch }}.sql"
```

### Example 3: Disable Firewall Management

Edit `roles/mariadb_server/defaults/main.yml`:

```yaml
mariadb_manage_firewall: false
```

Or use extra vars:

```bash
ansible-playbook -i inventory deploy_mariadb_site.yml -e "mariadb_manage_firewall=false"
```

### Example 4: Use Ansible Vault for Passwords

Create an encrypted vault file:

```bash
ansible-vault create group_vars/db_servers/vault.yml
```

Add passwords:

```yaml
mariadb_root_password: "YourEncryptedPassword"
mariadb_general_user_password: "AnotherEncryptedPassword"
```

Then run playbook with vault:

```bash
ansible-playbook -i inventory deploy_mariadb_site.yml --ask-vault-pass
```

## Best Practices

- **Use Ansible Vault**: Always encrypt passwords in production environments:

  ```bash
  ansible-vault encrypt_string 'YourPassword' --name 'mariadb_root_password'
  ```

- **Test First**: Use `--check` mode to preview changes:

  ```bash
  ansible-playbook -i inventory deploy_mariadb_site.yml --check
  ```

- **Validate Configuration**: After deployment, verify MariaDB is working:

  ```bash
  # Connect to MariaDB
  mysql -u root -p

  # Check databases
  SHOW DATABASES;

  # Check users
  SELECT user, host FROM mysql.user;
  ```

- **Verify Backup**: Check backup file was created:

  ```bash
  ls -lh /root/webapp_db_initial_backup.sql
  ```

- **Backup Configuration**: Consider backing up MariaDB configuration before changes
- **Test in Non-Production**: Test MariaDB configurations in non-production environments first
- **Document Changes**: Document database configuration changes for audit and troubleshooting

## Troubleshooting

### Playbook fails with "Unsupported operating system"

- Verify target host OS is supported (RHEL 7/8/9, CentOS 7/8/9)
- Check `ansible_distribution` and `ansible_distribution_major_version` facts
- Remove any unsupported hosts from the inventory

### "firewalld service is not available"

- **Disable firewall management** by setting:

  - `mariadb_manage_firewall: false` (in `roles/mariadb_server/defaults/main.yml`)
- Or install and enable firewalld:

  ```bash
  yum install firewalld
  systemctl enable --now firewalld
  ```

### "mariadb service fails to start"

- **Check service status:**

  ```bash
  systemctl status mariadb
  journalctl -u mariadb
  ```

- **Check MariaDB logs:**

  ```bash
  tail -f /var/log/mariadb/mariadb.log
  ```

- **Verify data directory permissions:**

  ```bash
  ls -ld /var/lib/mysql
  ```

### "Cannot connect to MariaDB" or "Access denied"

- **Check if MariaDB is running:**

  ```bash
  systemctl status mariadb
  ```

- **Verify root password:**

  ```bash
  mysql -u root -p
  ```

- **Check .my.cnf file:**

  ```bash
  cat /root/.my.cnf
  ```

- **Verify socket path:**

  ```bash
  ls -l /var/lib/mysql/mysql.sock
  ```

### "community.mysql collection not found" or "couldn't resolve module/action 'community.mysql.mysql_user'"

This error occurs when the `community.mysql` collection is not installed on the Ansible control node.

**Solution 1: Use Installation Script (Recommended for AAP)**

This project includes an installation script that installs the collection locally:

```bash
chmod +x install_collection.sh
./install_collection.sh
```

This will install the collection in the `collections/` directory within the project, which is ideal for AAP environments.

**Solution 2: Install using ansible-galaxy (Global Installation)**

```bash
ansible-galaxy collection install community.mysql
```

**Solution 3: Install from requirements.yml**

This project includes a `requirements.yml` file. Install using:

```bash
ansible-galaxy collection install -r requirements.yml
```

Or install to local directory:

```bash
ansible-galaxy collection install -r requirements.yml -p ./collections
```

**Verify Installation**

After installation, verify the collection is available:

```bash
# For local installation (in project directory)
ansible-galaxy collection list -p ./collections | grep community.mysql

# For global installation
ansible-galaxy collection list | grep community.mysql
```

Expected output:

```
community.mysql    <version>
```

**Check Collection Path**

If the collection is still not found, check where Ansible is looking for collections:

```bash
ansible-config dump | grep COLLECTIONS_PATHS
```

Ensure the collection is installed in one of the paths listed.

**For AAP (Ansible Automation Platform) Users**

If you're using AAP, the recommended approach is:

1. **Use the installation script** (`./install_collection.sh`) to install the collection in the project directory
2. The collection will be in `collections/` directory within your AAP project
3. AAP will automatically sync the collection when the project is synced
4. Alternatively, you can use a `requirements.yml` file and let AAP sync it automatically

Check your AAP project settings for collection management options.

### "python3-PyMySQL not found"

- The playbook automatically installs `python3-PyMySQL`, but if it fails:

  ```bash
  # RHEL 8/9
  dnf install python3-PyMySQL

  # RHEL 7
  yum install python3-PyMySQL
  ```

- Or install via pip:

  ```bash
  pip3 install PyMySQL
  ```

### "Database/user already exists"

- This is normal and non-fatal. Ansible is idempotent and will skip creation if resources already exist
- To force recreation, you may need to manually remove existing resources first

### "Backup file not created"

- **Check backup directory permissions:**

  ```bash
  ls -ld /root
  ```

- **Verify database exists:**

  ```bash
  mysql -u root -p -e "SHOW DATABASES;"
  ```

- **Check MariaDB user permissions:**

  ```bash
  mysql -u root -p -e "SHOW GRANTS FOR 'root'@'localhost';"
  ```

## Example Output

### Deploy MariaDB Server

```
PLAY [Deploy and configure MariaDB server] ******************************

TASK [Validate OS compatibility] **************************************
ok: [servera.example.com] => {
    "msg": "OS compatibility check passed: RedHat 8"
}

TASK [mariadb_server : Install MariaDB packages] **********************
changed: [servera.example.com]

TASK [mariadb_server : Start and enable MariaDB service] *************
changed: [servera.example.com]

TASK [mariadb_server : Set root password for all root@host variants] *
changed: [servera.example.com] => (item=localhost)
changed: [servera.example.com] => (item=127.0.0.1)
changed: [servera.example.com] => (item=::1)
changed: [servera.example.com] => (item=servera.example.com)

TASK [mariadb_server : Create .my.cnf file] **************************
changed: [servera.example.com]

TASK [mariadb_server : Delete anonymous users] ***********************
changed: [servera.example.com]

TASK [mariadb_server : Delete test database] *************************
changed: [servera.example.com]

TASK [mariadb_server : Create new database] **************************
changed: [servera.example.com]

TASK [mariadb_server : Create general user] **************************
changed: [servera.example.com]

TASK [mariadb_server : Grant privileges to general user] ************
changed: [servera.example.com]

TASK [mariadb_server : Dump database to backup file] *****************
changed: [servera.example.com]

TASK [mariadb_server : Ensure firewall allows MariaDB service] ******
changed: [servera.example.com]

PLAY RECAP ****************************************************************
servera.example.com      : ok=13   changed=12   unreachable=0    failed=0
```

## Role Design Philosophy

This solution uses Ansible Role architecture with the following design principles:

1. **Modularity**: Single role for complete MariaDB deployment
2. **Security First**: Automatic security hardening and password protection
3. **Idempotency**: All tasks are idempotent and safe to run multiple times
4. **Reusability**: Role can be used in any playbook or project
5. **Maintainability**: Easy to update and extend
6. **Best Practices**: FQDN modules, OS compatibility checks, error handling
7. **Configuration Flexibility**: Customizable via defaults and Ansible Vault

## Security Considerations

- **Root Access Required**: This playbook requires root privileges to modify MariaDB configurations
- **Password Security**: 
  - Passwords are protected with `no_log: true` to prevent exposure in logs
  - **Always use Ansible Vault** for passwords in production environments
  - Never commit plaintext passwords to version control
- **Database Service**: MariaDB is a critical service - test thoroughly before production
- **Firewall Rules**: Playbooks automatically configure firewalld - review rules for your environment
- **Backup Security**: Backup files contain database data - ensure proper file permissions
- **Test First**: Always test in non-production environments
- **Backup Configuration**: Consider backing up MariaDB configuration before changes

## Author

GCG AAP SSA Team + v3.01 20260217

## License

📜 License Type: End User License Agreement (EULA)
🔒 Authorization: Subscription-based License

