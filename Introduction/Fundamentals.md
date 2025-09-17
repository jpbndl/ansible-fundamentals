# Ansible

An open-source automation platform that simplifies IT infrastructure management.

## Ansible Configuration Files

Default configuration file is located at ***/etc/ansible/ansible.cfg***

```cfg
[defaults]
log_path        = /var/log/ansible.log
gathering       = implicit
...

[inventory]
enable_plugins  = host_list, virtualbox, yaml
....

[privilege_escalation]

[paramiko_connection]

[ssh_connection]

[persistent_connection]
```

### How to set up config files independently?

> **Note:** If you're unfamiliar with Linux folder structure, refer to [Linux Fundamentals](https://github.com/jpbndl/linux-administration/blob/main/Introduction/Fundamentals.md)

For different applications or setups requiring independent configurations, create project-specific `ansible.cfg` files:

- **/opt/web-playbooks/ansible.cfg**
- **/opt/db-playbooks/ansible.cfg**
- **/opt/network-playbooks/ansible.cfg**

What if you have all of them configured? The precedence is shown below: 

**Configuration Precedence (highest to lowest):**
1. `ANSIBLE_CONFIG` environment variable
2. `ansible.cfg` in current directory
3. `~/.ansible.cfg` in user's home directory
4. `/etc/ansible/ansible.cfg` system-wide default

**Configuration Variables**
- Non-presistent
    ```bash
        ANSIBLE_GATHERING explicit ansible-playbook playbook.yml
    ```
- Persistent throughout shell session
    ```bash
        export ANSIBLE_GATHERING=explicit
        ansible-playbook playbook.yml
    ```
- Persistent accross different shell and users
    ```bash
        /opt/web-playbook/ansible.cfg
        gethering = explicit
    ```

**Benefits:**
- **Isolation**: Each project uses its own settings
- **Portability**: Configuration travels with the project
- **Team consistency**: All team members use same settings
- **Environment-specific**: Different settings for dev/staging/prod

### View Configuration

To find out what the different configuration are, use the following command:

- **ansible-config list** - Lists all configurations
- **ansible-config view** - Shows the current config file
- **ansible-config dump** - Shows the current settings

Example:
```bash
    export ANSIBLE_GATHERING=explicit
    ansible-config dump | grep GATHERING
```

## Ansible Playbooks

Playbooks are Ansible's configuration, deployment, and orchestration language. They describe a set of steps to be executed on remote machines to achieve a desired state.

**Key Characteristics:**
- **Declarative**: Define what you want, not how to do it
- **Idempotent**: Safe to run multiple times
- **Human-readable**: Written in simple YAML syntax
- **Reusable**: Can be version controlled and shared

**Structure:**
- **Plays**: Map hosts to tasks
- **Tasks**: Individual units of work using modules
- **Handlers**: Tasks triggered by changes
- **Variables**: Dynamic values for customization

### YAML

All Ansible playbooks are written in YAML (YAML Ain't Markup Language) - a human-readable data serialization standard.

#### Key-Value Pairs
Simple name-value associations using colon and space:
```yaml
name: web-server
port: 80
enabled: true
version: 2.4.1
```

#### Arrays/Lists (Ordered)
Ordered collections using hyphens:
```yaml
packages:
  - nginx
  - git
  - curl

# Inline format
ports: [80, 443, 8080]
```

#### Dictionary/Maps (Unordered)
Nested key-value structures:
```yaml
server:
  name: web01
  ip: 192.168.1.10
  specs:
    cpu: 4
    memory: 8GB
    storage: 100GB
```

#### Sample Ansible Playbook
```yaml
- name: Install and configure web server
  hosts: webservers
  become: yes
  vars:
    packages:
      - nginx
      - git
    web_port: 80
  
  tasks:
    - name: Install packages
      package:
        name: "{{ item }}"
        state: present
      loop: "{{ packages }}"
    
    - name: Start nginx service
      service:
        name: nginx
        state: started
        enabled: yes
      notify: restart nginx
  
  handlers:
    - name: restart nginx
      service:
        name: nginx
        state: restarted
```
    