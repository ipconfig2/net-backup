# Network Programmability and Backup Automation

This project automates network device backups using Ansible. It supports both Cisco and Juniper devices and performs regular configuration and operational data backups. Additionally, the project integrates with GitHub for version control, pushing backup files to a repository.

## Prerequisites

Before you begin, ensure that you have the following installed:

- **Python** (version 3.6 or later)
- **Ansible** (version 2.9 or later)

You will also need to install the following Ansible collections:

```bash
ansible-galaxy collection install cisco.ios junipernetworks.junos ansible.posix ansible.netcommon
ansible-galaxy collection install community.general 
```

## Network Device Setup

### Cisco Devices
- SSH must be enabled.

### Juniper Devices
- Ensure SSH and Netconf services are enabled on each device.

For Juniper devices, make sure to configure the following settings:

```bash
set system services ssh root-login allow
set system services netconf ssh
```

For older Juniper devices, use this command instead:

```bash
set system services netconf rfc-compliant
```

## Configuration

### .ansible.cfg

Ensure that your `~/.ansible.cfg` is configured as follows:

```ini
[defaults]
host_key_checking = False
vault_password_file = ~/ansible-netauto/.ansible_vault_pass.txt
inventory = ~/ansible-netauto/backup_yaml_files/inventory.yml
```

### Ansible Vault

For encrypted inventory files, use Ansible Vault:

```bash
ansible-vault encrypt inventory.yml --vault-password-file ~/ansible-netauto/.ansible_vault_pass.txt
ansible-vault view inventory.yml --vault-password-file ~/ansible-netauto/.ansible_vault_pass.txt
ansible-vault edit inventory.yml --vault-password-file ~/ansible-netauto/.ansible_vault_pass.txt
```

To run playbooks with vault encryption:

```bash
ansible-playbook playbook.yml --ask-vault-pass
```

## Project Structure

### Playbooks

#### Backup Configuration and Show Commands

- **Cisco Routers & Switches**: The playbooks for Cisco devices back up configurations and run various show commands, saving their outputs for future reference.

- **Juniper Devices**: Juniper devices are backed up using netconf connection with show configuration and other operational data commands.

- **Backup Files**: Backup files are stored in a timestamped directory on the local machine under `~/network-programmability/backups/`.

- **GitHub Integration**: After each backup, files are automatically committed and pushed to a GitHub repository to ensure version control.

### Example Playbook Tasks

#### Cisco Router Configuration Backup

```yaml
- name: Backup IOS config
  cisco.ios.ios_command:
    commands: show run
  register: config
```

#### Juniper Configuration Backup

```yaml
- name: Run SHOW Config command for Juniper
  junipernetworks.junos.junos_command:
    commands:
      - show configuration
  register: juniper_config
```

#### GitHub Backup

After backups are completed, the playbook stages and pushes changes to GitHub:

```yaml
- name: Push backup to GitHub
  git:
    repo: 'https://github.com/username/network-backups.git'
    dest: '~/network-programmability/backups'
    version: 'main'
    state: present
    push: yes
```

## Running the Playbook

To run the playbook, use the following command:

```bash
ansible-playbook backup_playbook.yml
```

You may also need to provide your Ansible Vault password for encrypted files:

```bash
ansible-playbook backup_playbook.yml --ask-vault-pass
```

## Backup File Structure

Backup files will be saved in the following structure:

```
~/network-programmability/backups/
    └── <device_name>/
        ├── config_backup_YYYY-MM-DD_HH-MM-SS.txt
        ├── show_command_output_YYYY-MM-DD_HH-MM-SS.txt
        └── <other_backup_files>.txt
```

## GitHub Repository

After each backup, the backup files are pushed to a GitHub repository for version control. Make sure to replace `username` with your GitHub username in the repository URL:

```yaml
repo: 'https://github.com/username/network-backups.git'
```

## License

This project is licensed under the MIT License - see the LICENSE file for details.
