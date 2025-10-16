# Ansible Role: Netdata Agent

[![GitLab](https://img.shields.io/badge/GitLab-babidi34-orange?logo=gitlab)](https://gitlab.com/babidi34/ansible-role-netdata-agent)

This Ansible role installs and configures [Netdata Agent](https://www.netdata.cloud/), a real-time performance monitoring tool for Linux systems, with optional Netdata Cloud integration.

## 📦 Features

- **Smart Installation**: Checks for existing service to avoid redundant installations
- **Flexible Configuration**: Manages metrics retention and web server mode
- **Cloud Integration**: Automated claim/reclaim process for Netdata Cloud
- **Idempotent Design**: All tasks are designed to be safely rerunnable
- **Service Management**: Automatic restarts after configuration changes

## 📂 Role Structure

```bash
ansible-role-netdata-agent/
├── tasks/
│   ├── main.yml          # Main orchestration
│   ├── install.yml       # Conditional installation
│   ├── configure.yml     # Configuration deployment
│   └── claim.yml         # Netdata Cloud integration
├── handlers/
│   └── main.yml          # Restart handler
├── templates/
│   └── netdata.conf.j2   # Jinja2 configuration template
├── vars/
│   └── main.yml          # Default variables
└── README.md             # Documentation
```

## 🔧 Variables

### Main Variables

| Variable                     | Description                                                                 | Default Value                     | Required |
|------------------------------|-----------------------------------------------------------------------------|-----------------------------------|----------|
| `claim_token`                | Authentication token for Netdata Cloud                                     | `XXXXX`                           | Yes*     |
| `claim_rooms`                | Room IDs for Netdata Cloud                                                   | `XXXXX`                           | Yes*     |
| `claim_url`                  | Netdata Cloud service URL                                                   | `https://app.netdata.cloud`       | No       |
| `reclaim`                    | Force node reclaiming (true/false)                                          | `false`                           | No       |
| `dbengine_multihost_disk_space` | Disk space for metrics storage (MiB)                                     | `2048`                            | No       |
| `web_mode`                   | Web server mode (`none` to disable, `static-threaded` to enable local dashboard) | `none`                      | No       |

*Required only for Netdata Cloud integration

### System Variables Used

- `ansible_hostname`: Automatically used for hostname configuration in Netdata

## 🚀 Usage

### 1. Installation

```bash
ansible-galaxy install git+https://gitlab.com/babidi34/ansible-role-netdata-agent.git
```

Or via `requirements.yml`:

```yaml
---
roles:
  - name: babidi34.netdata_agent
    src: https://gitlab.com/babidi34/ansible-role-netdata-agent.git
    scm: git
    version: main
```

### 2. Example Playbook

```yaml
---
- name: Deploy Netdata Agent with Cloud integration
  hosts: all
  become: true
  roles:
    - role: babidi34.netdata_agent
      vars:
        claim_token: "your_netdata_cloud_token"
        claim_rooms: "your_room_id"
        dbengine_multihost_disk_space: 4096
        web_mode: "static-threaded"  # Enables local dashboard
```

### 3. Execution

```bash
ansible-playbook -i your_inventory.yml netdata.yml
```

## 🔄 Netdata Cloud Integration

The role automatically handles:

1. **First Installation**:
   - Automatic node claiming to Netdata Cloud
   - Creates `/var/lib/netdata/cloud.d/claimed_id` file

2. **Reinstallation**:
   - If `reclaim: true`, generates new ID using `uuidgen`
   - Requires `util-linux` package for `uuidgen`

3. **Verifications**:
   - Detects already claimed nodes
   - Handles errors if `uuidgen` is missing

## ⚙️ Technical Optimizations

1. **Conditional Installation**:
   ```yaml
   - name: Check if Netdata service is already installed
     stat:
       path: /lib/systemd/system/netdata.service
     register: netdata_service
   ```

2. **Idempotent Configuration**:
   - Uses Jinja2 templates with change notifications
   - Conditional service restarts

3. **Dependency Management**:
   - Automatic verification of `uuidgen` for reclaim operations

## 🛠️ Troubleshooting

### Common Errors

| Error | Probable Cause | Solution |
|-------|----------------|----------|
| `has no attribute 'hostname'` | Missing variable in old template | Fixed by using `ansible_hostname` |
| Claim failure | Invalid token/room | Verify `claim_token` and `claim_rooms` variables |
| Reclaim failure | Missing `uuidgen` | Install `util-linux` (`apt install util-linux`) |
| Service not starting | Dependency issues | Check logs with `journalctl -u netdata` |

### Useful Commands

```bash
# Check service status
systemctl status netdata

# View real-time logs
journalctl -u netdata -f

# Check generated configuration
cat /etc/netdata/netdata.conf

# Test Cloud connection
netdata-claim.sh --test
```

## 📝 Complete Example with Inventory

`inventory.ini`:
```ini
[servers]
my-server ansible_host=192.168.1.10

[servers:vars]
claim_token=my_secret_token
claim_rooms=my_room_id
dbengine_multihost_disk_space=4096
```

`netdata.yml`:
```yaml
---
- hosts: servers
  roles:
    - babidi34.netdata_agent
```

## 📋 Changelog

### Recent Improvements
- Added service existence check before installation
- Fixed host variable handling
- Optimized restart handlers
- Enhanced variable documentation

## 🤝 Contributing

Contributions are welcome! Please open a Merge Request on [GitLab](https://gitlab.com/babidi34/ansible-role-netdata-agent).

## 📄 License

[MIT](https://choosealicense.com/licenses/mit/)

## 👤 Maintainer

[Karim (babidi34)](https://gitlab.com/babidi34)
