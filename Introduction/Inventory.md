# Ansible Inventory

Ansible can work with one or multiple systems in your infrastructure at the same time by establishing connectivity to those servers. Unlike other agent tool, Ansible does not require you to install an agent in the target system. Information about the target systems are stored in an **inventory** file.

#### Default Inventory File
Default inventory file is stored in etc/Ansible/host

#### Connection
- **SSH** - Linux
- **PowerShell** - Windows

## Sample Inventory File

### INI Format
```ini
# Basic host list
web1.example.com
web2.example.com
iamanalias db1.example.com

# Grouped hosts
[webservers]
web1.example.com
web2.example.com

[databases]
db1.example.com
db2.example.com

# Hosts with variables
[webservers]
web1.example.com ansible_host=192.168.1.10 ansible_user=admin
web2.example.com ansible_host=192.168.1.11 ansible_user=admin

# Group variables
[webservers:vars]
ansible_ssh_port=22
ansible_python_interpreter=/usr/bin/python3
```

### YAML Format
```yaml
all:
  children:
    webservers:
      hosts:
        web1.example.com:
          ansible_host: 192.168.1.10
        web2.example.com:
          ansible_host: 192.168.1.11
      vars:
        ansible_user: admin
    databases:
      hosts:
        db1.example.com:
          ansible_host: 192.168.1.20
```

#### Why do we need different formats?
- INI format is simpler and easier to read for small inventories.
- YAML format is more flexible and better suited for complex inventories, supporting nested groups and advanced configuration options.YAML format