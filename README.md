# RHEL oVirt Template Builder

[![Build Status](https://travis-ci.org/oatakan/ansible-role-rhel-ovirt-template.svg?branch=master)](https://travis-ci.org/oatakan/ansible-role-rhel-ovirt-template)
[![Ansible Galaxy](https://img.shields.io/badge/ansible--galaxy-rhel_ovirt_template-blue.svg)](https://galaxy.ansible.com/oatakan/rhel_ovirt_template)

An Ansible role that automates the creation of RHEL/CentOS VM templates from ISO files on oVirt/RHV platforms. This role can be integrated into CI/CD pipelines for automated template building and management.

## Features

- **Multi-version Support**: RHEL 7, 8, 9, and 10
- **Automated Template Creation**: Build templates directly from ISO files
- **Security Hardening**: Built-in security configurations and SSH hardening
- **Cloud Integration**: Cloud-init ready templates
- **Flexible Configuration**: Customizable VM specifications and network settings
- **CI/CD Ready**: Perfect for automation pipelines
- **Template Management**: Create, update, and remove templates with force options

## Supported Distributions

| Distribution | Status | Notes |
|-------------|--------|--------|
| RHEL 7.x | ✅ Supported | Legacy support |
| RHEL 8.x | ✅ Supported | Stable |
| RHEL 9.x | ✅ Supported | Recommended |
| RHEL 10.x | ✅ Supported | Latest with modern features |
| CentOS 7/8 | ✅ Supported | Community distributions |
| CentOS Stream 9 | ✅ Supported | Rolling release |

## Prerequisites

### Control Machine Requirements

Install the following packages on your Ansible control machine:

```bash
# RHEL/CentOS/Fedora
sudo dnf install mkisofs genisoimage

# Ubuntu/Debian
sudo apt-get install mkisofs genisoimage
```

### oVirt/RHV Environment Setup

1. **Enable qemu_cmdline Hook**: Required for attaching multiple ISO files
   ```bash
   # Follow the official documentation:
   # https://www.ovirt.org/develop/developer-guide/vdsm/hook/qemucmdline.html
   ```

2. **Upload Installation Media**: Ensure your RHEL/CentOS ISO file is uploaded to an ISO domain on your oVirt/RHV environment

3. **Network Configuration**: Prepare a network with static IP allocation for template creation

## Installation

### From Ansible Galaxy

```bash
ansible-galaxy install oatakan.rhel_ovirt_template
```

### From Git Repository

```bash
git clone https://github.com/oatakan/ansible-role-rhel-ovirt-template.git
cd ansible-role-rhel-ovirt-template
```

## Dependencies

This role requires the following Ansible collections:

```yaml
collections:
  - ovirt.ovirt
```

Install dependencies:

```bash
ansible-galaxy collection install ovirt.ovirt
```

Additional role dependencies (automatically handled):

- `oatakan.rhn` - Red Hat subscription management
- `oatakan.rhel_upgrade` - System updates
- `oatakan.rhel_template_build` - Template preparation

## Quick Start

### Basic Template Creation

```yaml
---
- name: Create RHEL 10 template
  hosts: localhost
  gather_facts: false
  connection: local
  vars:
    # Authentication (use environment variables or vault)
    ovirt_url: "https://ovirt-engine.example.com/ovirt-engine/api"
    ovirt_username: "admin@internal"
    ovirt_password: "{{ vault_ovirt_password }}"
    
    # Template configuration
    template_vm_name: "rhel10-template-v1"
    distro_name: rhel10
    iso_file_name: "rhel-10.0-x86_64-dvd.iso"
    
    # VM specifications
    template_vm_memory: 4096
    template_vm_cpu: 2
    template_vm_root_disk_size: 20
    
    # Network configuration
    template_vm_ip_address: "192.168.1.100"
    template_vm_netmask: "255.255.255.0"
    template_vm_gateway: "192.168.1.1"
    template_vm_dns_servers:
      - "8.8.8.8"
      - "8.8.4.4"
    
    # Security
    local_account_username: "ansible"
    local_account_password: "{{ vault_ansible_password }}"
    local_administrator_password: "{{ vault_root_password }}"
    
  roles:
    - oatakan.rhel_ovirt_template
```

## Configuration

### Required Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `ovirt_datacenter` | oVirt datacenter name | `"Default"` |
| `ovirt_cluster` | oVirt cluster name | `"production"` |
| `ovirt_data_domain` | Data storage domain | `"data_domain"` |
| `iso_file_name` | ISO file name | `"rhel-10.0-x86_64-dvd.iso"` |
| `template_vm_ip_address` | Static IP for template VM | `"192.168.1.100"` |
| `local_account_password` | Ansible user password | `"secure_password"` |
| `local_administrator_password` | Root password | `"secure_root_password"` |

### Distribution Configuration

| Distribution | `distro_name` | Recommended ISO |
|-------------|---------------|-----------------|
| RHEL 10 | `rhel10` | `rhel-10.0-x86_64-dvd.iso` |
| RHEL 9 | `rhel9` | `rhel-9.3-x86_64-dvd.iso` |
| RHEL 8 | `rhel8` | `rhel-8.9-x86_64-dvd.iso` |
| RHEL 7 | `rhel7` | `rhel-server-7.9-x86_64-dvd.iso` |
| CentOS Stream 9 | `rhel9` | `CentOS-Stream-9-latest-x86_64-dvd1.iso` |

### VM Specifications

```yaml
# VM Hardware Configuration
template_vm_memory: 4096          # Memory in MB
template_vm_cpu: 2               # Number of CPU cores
template_vm_root_disk_size: 20   # Root disk size in GB
template_vm_root_disk_format: cow # Disk format (cow/raw)
template_vm_efi: false           # Enable UEFI boot

# Network Configuration
template_vm_network_name: "ovirtmgmt"
template_vm_ip_address: "192.168.1.100"
template_vm_netmask: "255.255.255.0"
template_vm_gateway: "192.168.1.1"
template_vm_domain: "example.com"
template_vm_dns_servers:
  - "8.8.8.8"
  - "8.8.4.4"
```

### Security Configuration

```yaml
# Security Settings
template_selinux_enabled: false          # Enable SELinux
permit_root_login_with_password: false   # Allow root SSH login
install_updates: true                    # Install latest updates

# User Configuration
local_account_username: "ansible"
local_account_password: "{{ vault_password }}"
local_administrator_password: "{{ vault_root_password }}"
```

### Advanced Options

```yaml
# Template Management
template_force: false          # Overwrite existing template
export_ovf: false             # Export template to export domain
template_convert_timeout: 600  # Template conversion timeout

# Cleanup Options
remove_vm_on_error: true      # Remove VM if creation fails
```

## Usage Examples

### Example 1: RHEL 10 Production Template

```yaml
---
- name: Create production RHEL 10 template
  hosts: localhost
  gather_facts: false
  connection: local
  vars:
    # Template identification
    template_vm_name: "rhel10-prod-template"
    distro_name: rhel10
    iso_file_name: "rhel-10.0-x86_64-dvd.iso"
    
    # Production specifications
    template_vm_memory: 8192
    template_vm_cpu: 4
    template_vm_root_disk_size: 50
    template_vm_efi: true
    
    # Security hardening
    template_selinux_enabled: true
    permit_root_login_with_password: false
    install_updates: true
    
    # Infrastructure
    ovirt_datacenter: "datacenter1"
    ovirt_cluster: "production"
    ovirt_data_domain: "production_storage"
    
    # Network
    template_vm_network_name: "production_network"
    template_vm_ip_address: "10.0.1.100"
    template_vm_netmask: "255.255.255.0"
    template_vm_gateway: "10.0.1.1"
    template_vm_dns_servers:
      - "10.0.1.10"
      - "10.0.1.11"
      
  roles:
    - oatakan.rhel_ovirt_template
```

### Example 2: Development Environment

```yaml
---
- name: Create development template
  hosts: localhost
  gather_facts: false
  connection: local
  vars:
    template_vm_name: "rhel9-dev-template"
    distro_name: rhel9
    iso_file_name: "rhel-9.3-x86_64-dvd.iso"
    
    # Minimal development specs
    template_vm_memory: 2048
    template_vm_cpu: 2
    template_vm_root_disk_size: 20
    
    # Development-friendly settings
    template_selinux_enabled: false
    permit_root_login_with_password: true
    install_updates: false
    
  roles:
    - oatakan.rhel_ovirt_template
```

### Example 3: Template Removal

```yaml
---
- name: Remove template
  hosts: localhost
  gather_facts: false
  connection: local
  vars:
    template_vm_name: "old-template"
    role_action: deprovision
    
  roles:
    - oatakan.rhel_ovirt_template
```

## CI/CD Integration

### Jenkins Pipeline Example

```groovy
pipeline {
    agent any
    
    environment {
        OVIRT_URL = credentials('ovirt-url')
        OVIRT_USERNAME = credentials('ovirt-username')
        OVIRT_PASSWORD = credentials('ovirt-password')
    }
    
    stages {
        stage('Build RHEL Template') {
            steps {
                ansiblePlaybook(
                    playbook: 'template-build.yml',
                    inventory: 'localhost,',
                    extraVars: [
                        template_vm_name: "rhel10-${BUILD_NUMBER}",
                        distro_name: 'rhel10',
                        template_force: true
                    ]
                )
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
    }
}
```

### GitLab CI Example

```yaml
build_template:
  stage: build
  script:
    - ansible-playbook template-build.yml -i localhost, 
      --extra-vars "template_vm_name=rhel10-${CI_PIPELINE_ID}"
  only:
    - master
  tags:
    - ansible
```

## Troubleshooting

### Common Issues

1. **ISO File Not Found**
   ```
   Error: iso file could not be found on the data domain
   ```
   **Solution**: Verify the ISO file is uploaded to the correct storage domain

2. **Network Connectivity Issues**
   ```
   Error: waiting for server to come online
   ```
   **Solution**: Check static IP configuration and network connectivity

3. **Authentication Failures**
   ```
   Error: SSO token authentication failed
   ```
   **Solution**: Verify oVirt credentials and URL

4. **Template Already Exists**
   ```
   Error: Existing template found on ovirt/rhv
   ```
   **Solution**: Set `template_force: true` to overwrite existing templates

### Debug Mode

Enable verbose logging:

```bash
ansible-playbook -vvv playbook.yml
```

### Log Files

Check the following log files on the template VM during creation:

- `/var/log/ks-post.log` - Kickstart post-installation log
- `/var/log/template-build.log` - Template preparation log
- `/var/log/anaconda/` - Installation logs

## Security Considerations

### Default Security Features

- **SSH Hardening**: Modern cipher suites and authentication methods
- **User Management**: Non-root user with sudo access
- **SELinux**: Optional enforcement with proper contexts
- **Firewall**: Minimal open ports configuration
- **Updates**: Latest security patches (when enabled)

### Security Best Practices

1. **Use Ansible Vault** for sensitive variables:
   ```bash
   ansible-vault create vars/secrets.yml
   ```

2. **Disable root login** in production:
   ```yaml
   permit_root_login_with_password: false
   ```

3. **Enable SELinux** for production templates:
   ```yaml
   template_selinux_enabled: true
   ```

4. **Use strong passwords** and consider key-based authentication

## Contributing

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/new-feature`
3. Make your changes and test thoroughly
4. Commit your changes: `git commit -am 'Add new feature'`
5. Push to the branch: `git push origin feature/new-feature`
6. Submit a pull request

### Development Setup

```bash
git clone https://github.com/oatakan/ansible-role-rhel-ovirt-template.git
cd ansible-role-rhel-ovirt-template

# Install development dependencies
pip install ansible-lint yamllint

# Run syntax checks
ansible-playbook tests/test.yml -i tests/inventory --syntax-check

# Run linting
ansible-lint .
yamllint .
```

## Changelog

### v2.1.0 (Latest)
- ✅ Added RHEL 10 support with modern features
- ✅ Enhanced security configurations
- ✅ Improved cloud-init integration
- ✅ Updated documentation

### v2.0.0
- ✅ RHEL 9 support
- ✅ Modernized kickstart configurations
- ✅ Improved error handling

### v1.x
- ✅ Initial RHEL 7/8 support
- ✅ Basic template creation functionality

## License

MIT License - see [LICENSE](LICENSE) file for details.

## Author Information

**Orcun Atakan**  
Senior Solutions Architect, Red Hat

- GitHub: [@oatakan](https://github.com/oatakan)
- LinkedIn: [Orcun Atakan](https://linkedin.com/in/oatakan)

## Support

- **Issues**: [GitHub Issues](https://github.com/oatakan/ansible-role-rhel-ovirt-template/issues)
- **Documentation**: [Ansible Galaxy](https://galaxy.ansible.com/oatakan/rhel_ovirt_template)
- **Community**: [Red Hat Community](https://community.redhat.com)

---

> **Note**: This role is provided as an example and reference implementation. For production use, please review and customize the configurations according to your organization's security and operational requirements.
```

### 4. Update default configuration in `defaults/main.yml`

Change the default distro to rhel10 and update the ISO file name:

```yaml
distro_name: rhel10
iso_file_name: rhel-10.0-x86_64-dvd.iso
linux_ks_folder: rhel10
template_vm_name: rhel10-x64-v1
```

## Pull Request Description

### Summary

This PR adds support for RHEL 10 to the rhel_ovirt_template Ansible role, enabling users to build RHEL 10 VM templates from ISO files on oVirt/RHV.

### Changes Made

1. **Added RHEL 10 kickstart template** (`templates/rhel10/ks.cfg.j2`):
   - Based on modern RHEL 10 best practices
   - Includes enhanced security configurations
   - Uses dnf5 package manager
   - Implements modern systemd features (systemd-resolved, systemd-networkd)
   - Enhanced SSH security configurations
   - Cloud-init integration for template preparation
   - Optimized for VM template usage

2. **Updated configuration files**:
   - Added `rhel10` to `os_short_names` in `defaults/main.yml`
   - Updated README.md with RHEL 10 examples
   - Set RHEL 10 as default distro

### Key Features of RHEL 10 Support

- **Modern package management**: Uses dnf5 for improved performance
- **Enhanced security**: Includes modern crypto policies and SSH hardening
- **Cloud-ready**: Integrated cloud-init for better cloud/virtualization support
- **Systemd integration**: Leverages modern systemd features
- **Template optimization**: Includes cleanup and preparation steps for template usage

### Testing

- [ ] Syntax check with `ansible-playbook tests/test.yml -i tests/inventory --syntax-check`
- [ ] Template creation with RHEL 10 ISO
- [ ] Verify all security configurations are applied correctly
- [ ] Confirm template sealing and cleanup work properly

### Backward Compatibility

This change is fully backward compatible. Existing configurations for RHEL 7, 8, and 9 remain unchanged and functional.

## Files Changed

- `defaults/main.yml` - Added RHEL 10 configuration
- `templates/rhel10/ks.cfg.j2` - New RHEL 10 kickstart template
- `README.md` - Updated documentation with RHEL 10 examples

## How to Use

Set `distro_name: rhel10` in your playbook variables and provide a RHEL 10 ISO file to automatically use the new RHEL 10 kickstart configuration.