# Windows VM Health Check Ansible Playbook

A comprehensive Ansible playbook for monitoring and diagnosing the health of Windows virtual machines across multiple Windows versions including Windows Server 2016, 2019, 2022, Windows 10, and Windows 11.

## 🎯 Purpose

This playbook provides automated health monitoring for Windows VMs in enterprise environments. It performs comprehensive system checks using PowerShell and WMI queries, generating detailed health reports that can be integrated with monitoring systems or used for compliance reporting.

## ✨ Features

### System Monitoring
- **CPU Usage Analysis** - Real-time CPU utilization with configurable thresholds
- **Memory Assessment** - Physical and virtual memory usage monitoring
- **Disk Space Monitoring** - Multi-drive disk usage tracking with SMART data
- **Network Interface Status** - Network adapter state and connectivity checks
- **System Performance Counters** - Windows performance metrics analysis

### Service Health Checks
- **Windows Service Monitoring** - Critical service status verification
- **Process Monitoring** - Essential process availability checks
- **Registry Health** - Critical registry key validation
- **Event Log Analysis** - System and application event log scanning
- **Windows Update Status** - Patch level and update compliance checks

### Reporting & Alerting
- **JSON Health Reports** - Structured output for SIEM and monitoring tool integration
- **PowerShell Integration** - Native Windows diagnostic commands
- **WMI Query Results** - Deep system information gathering
- **Performance Baseline** - Historical comparison capabilities
- **Multi-environment Support** - Production, staging, and development configurations

## 🚀 Quick Start

### Prerequisites
- Ansible 8.0+ with Windows support installed on the control node
- PowerShell 5.1+ on target Windows systems
- WinRM configured and enabled on target systems
- Administrative privileges on target systems
- pywinrm Python package installed on control node

### Installation

1. **Clone the repository:**
   ```bash
   git clone https://github.com/keepithuman/windows-vm-health-check.git
   cd windows-vm-health-check
   ```

2. **Install dependencies:**
   ```bash
   ansible-galaxy install -r requirements.yml
   pip install -r requirements.txt
   ```

3. **Configure WinRM on Windows targets:**
   ```powershell
   # Run on each Windows target
   winrm quickconfig -y
   winrm set winrm/config/service/auth '@{Basic="true"}'
   winrm set winrm/config/service '@{AllowUnencrypted="true"}'
   ```

4. **Configure inventory:**
   Edit `inventory.yml` to add your Windows VMs:
   ```yaml
   production:
     hosts:
       win-server-01:
         ansible_host: 10.1.1.10
         ansible_user: Administrator
         ansible_password: "{{ vault_admin_password }}"
       win-web-01:
         ansible_host: 10.1.1.11
         ansible_user: Administrator
         ansible_password: "{{ vault_admin_password }}"
   ```

5. **Run the health check:**
   ```bash
   ansible-playbook -i inventory.yml windows-health-check.yml
   ```

## 📋 Configuration

### Inventory Variables

Configure health check parameters in `inventory.yml`:

```yaml
all:
  vars:
    # Connection settings
    ansible_connection: winrm
    ansible_winrm_transport: basic
    ansible_winrm_server_cert_validation: ignore
    
    # Performance thresholds
    disk_threshold_warning: 80      # Disk usage warning %
    disk_threshold_critical: 90     # Disk usage critical %
    memory_threshold_warning: 80    # Memory usage warning %
    memory_threshold_critical: 90   # Memory usage critical %
    cpu_threshold_warning: 80       # CPU usage warning %
    
    # Services to monitor
    check_services:
      - "Spooler"
      - "BITS"
      - "Windows Update"
      - "Windows Firewall"
      - "RPC Endpoint Mapper"
    
    # Processes to verify are running
    check_processes:
      - "explorer"
      - "winlogon"
      - "csrss"
    
    # Network ports to verify
    check_ports:
      - 3389  # RDP
      - 5985  # WinRM HTTP
      - 5986  # WinRM HTTPS
    
    # Event log checking
    event_log_hours: 24  # Hours to look back for errors
    
    # Behavior settings
    fail_on_critical: false  # Stop playbook on critical issues
```

### Environment-Specific Settings

Different environments can have different thresholds:

```yaml
production:
  vars:
    disk_threshold_warning: 70    # Stricter for production
    memory_threshold_warning: 75
    fail_on_critical: true        # Fail fast in production
    check_services:
      - "Spooler"
      - "BITS"
      - "Windows Update"
      - "IIS Admin Service"
      - "SQL Server"

development:
  vars:
    disk_threshold_warning: 85    # Relaxed for development
    fail_on_critical: false       # Never fail in dev
    event_log_hours: 8           # Less log history needed
```

## 🎮 Usage Examples

### Basic Health Check
```bash
# Check all configured Windows VMs
ansible-playbook -i inventory.yml windows-health-check.yml

# Check specific environment
ansible-playbook -i inventory.yml windows-health-check.yml --limit production

# Check specific host
ansible-playbook -i inventory.yml windows-health-check.yml --limit win-server-01
```

### Tag-Based Execution
```bash
# Only check performance metrics
ansible-playbook -i inventory.yml windows-health-check.yml --tags performance

# Only check services
ansible-playbook -i inventory.yml windows-health-check.yml --tags services

# Skip event log analysis
ansible-playbook -i inventory.yml windows-health-check.yml --skip-tags events
```

### Using Ansible Vault for Credentials
```bash
# Create encrypted password file
ansible-vault create group_vars/all/vault.yml

# Run with vault
ansible-playbook -i inventory.yml windows-health-check.yml --ask-vault-pass
```

## 📊 Health Report Format

The playbook generates comprehensive JSON reports in `C:\temp\health_reports\`:

```json
{
  "health_check": {
    "timestamp": "2025-06-30T21:30:00Z",
    "hostname": "WIN-SERVER-01",
    "status": "WARNING",
    "summary": {
      "warnings_count": 3,
      "critical_issues_count": 0
    }
  },
  "system_info": {
    "hostname": "WIN-SERVER-01",
    "os_name": "Microsoft Windows Server 2022 Standard",
    "os_version": "10.0.20348",
    "architecture": "AMD64",
    "domain": "CONTOSO.COM",
    "uptime_days": 30.5,
    "last_boot_time": "2025-05-31T10:30:00Z"
  },
  "performance_metrics": {
    "cpu": {
      "usage_percent": 25.8,
      "processor_count": 4
    },
    "memory": {
      "total_gb": 16.0,
      "available_gb": 8.2,
      "used_percent": 48.8,
      "page_file_usage_percent": 15.3
    },
    "disk_usage": [
      {
        "drive": "C:",
        "size_gb": 100.0,
        "free_gb": 25.5,
        "used_percent": 74.5
      }
    ]
  },
  "services": {
    "running_services_count": 156,
    "stopped_critical_services": [],
    "service_status": {
      "Spooler": "Running",
      "BITS": "Running",
      "Windows Update": "Stopped"
    }
  },
  "events": {
    "system_errors_24h": 2,
    "application_errors_24h": 1,
    "security_warnings_24h": 0
  },
  "issues": {
    "warnings": [
      "Windows Update service is stopped",
      "High CPU usage: 85.2%",
      "Found 3 system errors in last 24 hours"
    ],
    "critical": []
  }
}
```

## 🏷️ Available Tags

| Tag | Description |
|-----|-------------|
| `setup` | Create directories and prepare environment |
| `info` | Gather system information |
| `cpu` | CPU usage and processor checks |
| `memory` | Memory and page file analysis |
| `disk` | Disk space utilization checks |
| `network` | Network interface and connectivity checks |
| `services` | Windows service status verification |
| `processes` | Process monitoring and validation |
| `events` | Event log analysis |
| `registry` | Registry health checks |
| `updates` | Windows Update status checks |
| `performance` | All performance-related checks |
| `report` | Generate health reports |

## 🔧 Integration with Itential Automation Gateway (IAG)

This playbook is designed to work seamlessly with Itential Automation Gateway:

### Creating the IAG Repository
```bash
iagctl create repository windows-health-repo \
  --description "Windows VM Health Check Repository" \
  --url "https://github.com/keepithuman/windows-vm-health-check.git" \
  --reference main
```

### Creating the IAG Service
```bash
iagctl create service ansible-playbook windows-vm-health-check \
  --repository windows-health-repo \
  --playbook windows-health-check.yml \
  --inventory inventory.yml \
  --description "Comprehensive Windows VM health monitoring service" \
  --tag health-check \
  --tag monitoring \
  --tag windows
```

### Running via IAG
```bash
iagctl run service ansible-playbook windows-vm-health-check
```

## 🎛️ Customization

### Adding Custom Registry Checks

Extend the playbook with custom registry validations:

```yaml
- name: Check custom application registry keys
  win_reg_stat:
    path: HKLM:\SOFTWARE\MyCompany\MyApp
    name: Version
  register: app_registry
  tags: ['custom', 'registry']

- name: Evaluate application registry health
  set_fact:
    warnings: "{{ warnings + ['MyApp registry key missing or invalid'] }}"
  when: not app_registry.exists
  tags: ['custom', 'registry']
```

### Custom Performance Counters

Add specific Windows performance counter monitoring:

```yaml
- name: Check IIS performance counters
  win_shell: |
    Get-Counter "\Web Service(_Total)\Current Connections" | 
    Select-Object -ExpandProperty CounterSamples | 
    Select-Object -ExpandProperty CookedValue
  register: iis_connections
  when: "'IIS' in check_services"
  tags: ['custom', 'iis']
```

## 🔍 Troubleshooting

### Common Issues

1. **WinRM Connection Failures**
   ```bash
   # Test WinRM connectivity
   ansible windows -i inventory.yml -m win_ping
   
   # Check WinRM configuration
   winrm get winrm/config
   ```

2. **Authentication Issues**
   ```yaml
   # Use domain authentication
   ansible_user: "DOMAIN\\username"
   ansible_password: "{{ vault_password }}"
   
   # Or use UPN format
   ansible_user: "username@domain.com"
   ```

3. **PowerShell Execution Policy**
   ```powershell
   # On target Windows machines
   Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope LocalMachine
   ```

4. **Firewall Configuration**
   ```powershell
   # Allow WinRM through Windows Firewall
   netsh advfirewall firewall add rule name="WinRM-HTTP" dir=in localport=5985 protocol=TCP action=allow
   netsh advfirewall firewall add rule name="WinRM-HTTPS" dir=in localport=5986 protocol=TCP action=allow
   ```

### Debug Mode

Enable detailed logging:
```bash
ansible-playbook -i inventory.yml windows-health-check.yml -vvv
```

### Testing WinRM Configuration

```bash
# Test from Linux control node
python -c "import winrm; r = winrm.Session('http://TARGET_IP:5985/wsman', auth=('user', 'pass')); print(r.run_cmd('ipconfig'))"
```

## 📝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🤝 Support

- **Issues**: [GitHub Issues](https://github.com/keepithuman/windows-vm-health-check/issues)
- **Discussions**: [GitHub Discussions](https://github.com/keepithuman/windows-vm-health-check/discussions)
- **Documentation**: This README and inline code comments

## 🎉 Acknowledgments

- Built for the Ansible community
- Designed for enterprise Windows environments
- Optimized for Itential Automation Gateway integration
- Inspired by modern Windows administration practices

---

**Made with ❤️ for reliable Windows infrastructure monitoring**