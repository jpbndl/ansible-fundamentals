# Ansible Variables

Variables in Ansible are used to store dynamic values that can be reused throughout playbooks, roles, and tasks. They help in customizing the behavior of Ansible based on different environments or conditions.

## Sample Variables

### String Variables
```yaml
app_name: "my-web-app"
app_version: "1.2.3"
app_environment: "production"
```

### Numeric Variables
```yaml
max_connections: 100
port: 8080
timeout: 30
```

### Boolean Variables
```yaml
enable_ssl: true
debug_mode: false
auto_restart: true
```

### List Variables
```yaml
packages:
  - nginx
  - mysql-server
  - php-fpm

users:
  - alice
  - bob
  - charlie
```

### Dictionary Variables
```yaml
database:
  host: "localhost"
  port: 3306
  name: "myapp_db"
  user: "dbuser"

ssl_config:
  cert_path: "/etc/ssl/certs/app.crt"
  key_path: "/etc/ssl/private/app.key"
  protocols:
    - "TLSv1.2"
    - "TLSv1.3"
```

### Playbook vars Section
```yaml
---
- hosts: webservers
  vars:
    app_name: "my-web-app"
    app_version: "1.2.3"
    packages:
      - nginx
      - php-fpm
    database:
      host: "localhost"
      port: 3306
  tasks:
    - name: Install packages
      package:
        name: "{{ item }}"
        state: present
      loop: "{{ packages }}"
```

### Task-level vars
```yaml
- name: Configure application
  template:
    src: app.conf.j2
    dest: "/etc/{{ app_name }}/config.conf"
  vars:
    listen_port: 8080
    ssl_enabled: true
```

### Register Variables

Register variables capture the output of a task and store it in a variable for later use. When you use `register`, Ansible saves the task's result (stdout, stderr, return code, etc.) into the specified variable name. These variables are scoped to the specific host where the task runs - each host gets its own copy of the registered variable.

```yaml
- name: Check service status
  command: systemctl is-active nginx
  register: service_status
  ignore_errors: true

- name: Get disk usage
  shell: df -h /var/log
  register: disk_info

- name: Use registered variables
  debug:
    msg: |
      Service status: {{ service_status.stdout }}
      Return code: {{ service_status.rc }}
      Disk info: {{ disk_info.stdout_lines[1] }}

- name: Conditional based on register
  service:
    name: nginx
    state: started
  when: service_status.rc != 0
```

## Inventory Variables

### Host Variables
```ini
[webservers]
web1.example.com ansible_host=192.168.1.10 http_port=80
web2.example.com ansible_host=192.168.1.11 http_port=8080

[databases]
db1.example.com ansible_host=192.168.1.20 mysql_port=3306
```

### Group Variables
```ini
[webservers]
web1.example.com
web2.example.com

[webservers:vars]
app_user=www-data
log_level=info

[databases]
db1.example.com

[databases:vars]
backup_enabled=true
max_connections=200
```

### YAML Inventory Format
```yaml
all:
  children:
    webservers:
      hosts:
        web1.example.com:
          ansible_host: 192.168.1.10
          http_port: 80
        web2.example.com:
          ansible_host: 192.168.1.11
          http_port: 8080
      vars:
        app_user: www-data
        log_level: info
    databases:
      hosts:
        db1.example.com:
          ansible_host: 192.168.1.20
          mysql_port: 3306
      vars:
        backup_enabled: true
        max_connections: 200
```

## Jinja2 Templating

Jinja2 is a templating engine that Ansible uses to dynamically generate content. It allows you to embed variables, conditionals, and loops within templates and task parameters.

**Key Syntax:**
- `{{ variable }}` - Output variables
- `{% statement %}` - Control structures (if, for, etc.)
- `{# comment #}` - Comments (not rendered)

### Basic Variable Interpolation
```yaml
- name: Create config file
  copy:
    content: |
      server_name {{ ansible_hostname }}
      listen {{ http_port }}
      root /var/www/{{ app_name }}
    dest: /etc/nginx/sites-available/{{ app_name }}
```

### Conditional Statements
```yaml
- name: Template with conditions
  template:
    src: config.j2
    dest: /etc/app/config.conf
```

**config.j2 template:**
```jinja2
server_port={{ http_port }}
{% if ssl_enabled %}
ssl_cert={{ ssl_cert_path }}
ssl_key={{ ssl_key_path }}
{% endif %}

{% if debug_mode %}
log_level=debug
{% else %}
log_level=info
{% endif %}
```

### Loops in Templates
```jinja2
# Database hosts
{% for host in groups['databases'] %}
db_host_{{ loop.index }}={{ hostvars[host]['ansible_host'] }}
{% endfor %}

# Package list
{% for package in packages %}
install {{ package }}
{% endfor %}
```

### Filters
```yaml
- name: Using filters
  debug:
    msg: |
      Uppercase: {{ app_name | upper }}
      Default value: {{ undefined_var | default('fallback') }}
      Join list: {{ packages | join(', ') }}
      Random password: {{ ansible_date_time.epoch | password_hash('sha512') }}
```

## Variable Scope

Ansible variables have different scopes that determine their visibility and accessibility:

### Host Scope
Variables specific to individual hosts. Each host maintains its own copy.

```yaml
# Inventory host variables
web1.example.com http_port=80
web2.example.com http_port=8080

# Register variables (host-specific results)
- name: Check disk space
  shell: df -h /
  register: disk_usage  # Each host gets its own disk_usage

# Host facts
- debug:
    msg: "This host's IP: {{ ansible_default_ipv4.address }}"
```

### Play Scope
Variables available to all hosts within a specific play.

```yaml
---
- hosts: webservers
  vars:
    app_name: "my-app"     # Available to all webserver hosts
    app_version: "1.0"
  tasks:
    - name: All webservers can access these vars
      debug:
        msg: "Deploying {{ app_name }} v{{ app_version }}"

- hosts: databases
  vars:
    db_name: "mydb"        # Only available to database hosts
  tasks:
    - debug:
        msg: "{{ app_name }}"  # ERROR: app_name not in scope
```

### Global Scope
Variables accessible across all plays and hosts in the playbook.

```yaml
# Extra vars (command line: -e "env=prod")
# Group vars/all
# Inventory group variables for 'all' group

[all:vars]
company_name=MyCompany
environment=production

# These are available everywhere:
---
- hosts: webservers
  tasks:
    - debug: msg="{{ company_name }} - {{ environment }}"

- hosts: databases  
  tasks:
    - debug: msg="{{ company_name }} - {{ environment }}"
```

### Scope Access Example
```yaml
---
- hosts: webservers
  vars:
    play_var: "play level"           # Play scope
  tasks:
    - name: Set host variable
      set_fact:
        host_var: "host level"         # Host scope
    
    - name: Show variable scopes
      debug:
        msg: |
          Global: {{ environment }}     # Global scope
          Play: {{ play_var }}          # Play scope  
          Host: {{ host_var }}          # Host scope
          Host-specific: {{ ansible_hostname }}  # Host scope
```

## Variable Precedence

Ansible follows a specific order when multiple sources define the same variable (highest to lowest priority):

1. **Extra vars** (`-e` command line)
2. **Task vars** (in task definition)
3. **Block vars** (in block definition)
4. **Role and include vars**
5. **Play vars** (in playbook `vars:` section)
6. **Host facts** (gathered facts)
7. **Inventory host vars**
8. **Inventory group vars**
9. **Group vars/all**
10. **Role defaults**

```yaml
# Example: If same variable defined in multiple places
# Command: ansible-playbook -e "app_port=9000" playbook.yml
# Result: app_port=9000 (extra vars win)

- hosts: webservers
  vars:
    app_port: 8080  # Lower priority
  tasks:
    - name: Show port
      debug:
        msg: "Port: {{ app_port }}"  # Will show 9000
```

## Variable Association to Hosts

Ansible associates variables to hosts through multiple mechanisms:

### Host-specific Variables
```yaml
# Inventory host vars
web1.example.com ansible_host=192.168.1.10 custom_port=80
web2.example.com ansible_host=192.168.1.11 custom_port=8080

# Each host gets its own custom_port value
```

### Group Variables
```yaml
# All hosts in 'webservers' group inherit these vars
[webservers:vars]
app_user=www-data
log_level=info
```

### Accessing Other Host Variables
```yaml
- name: Access another host's variables
  debug:
    msg: "DB host IP: {{ hostvars['db1.example.com']['ansible_host'] }}"

- name: Loop through all database hosts
  debug:
    msg: "DB {{ item }}: {{ hostvars[item]['ansible_host'] }}"
  loop: "{{ groups['databases'] }}"
```

### Variable Scope Example
```yaml
---
- hosts: webservers
  vars:
    global_var: "available to all webserver hosts"
  tasks:
    - name: Host-specific task
      debug:
        msg: |
          Host: {{ inventory_hostname }}
          Global: {{ global_var }}
          Host-specific: {{ custom_port | default('not set') }}
      # Each host sees same global_var but different custom_port
```

## Magic Variables

Ansible provides special built-in variables that give access to system information and cross-host data.

### hostvars - Accessing Other Host Variables

The `hostvars` magic variable allows any host to access variables from other hosts in the inventory.

```yaml
# Inventory
[webservers]
web1.example.com db_host=192.168.1.10
web2.example.com db_host=192.168.1.11  
web3.example.com db_host=192.168.1.12

[databases]
db1.example.com mysql_port=3306 backup_enabled=true

# Playbook - web1 and web3 accessing web2's variables
---
- hosts: webservers
  tasks:
    - name: Set host-specific variable on web2
      set_fact:
        secret_key: "web2-secret-123"
      when: inventory_hostname == "web2.example.com"
    
    - name: Access web2's variables from other hosts
      debug:
        msg: |
          My hostname: {{ inventory_hostname }}
          Web2's db_host: {{ hostvars['web2.example.com']['db_host'] }}
          Web2's secret: {{ hostvars['web2.example.com']['secret_key'] | default('not set') }}
          Web2's IP: {{ hostvars['web2.example.com']['ansible_default_ipv4']['address'] }}
```

### groups - Group Magic Variable

The `groups` variable contains all inventory groups and their host lists.

```yaml
- hosts: webservers
  tasks:
    - name: Show all groups
      debug:
        msg: "Available groups: {{ groups.keys() | list }}"
    
    - name: List all webserver hosts
      debug:
        msg: "Webserver hosts: {{ groups['webservers'] }}"
    
    - name: Configure database connections
      template:
        src: db_config.j2
        dest: /etc/app/db.conf
      vars:
        db_servers: "{{ groups['databases'] }}"

# Template: db_config.j2
{% for db_host in groups['databases'] %}
db_server_{{ loop.index }}={{ hostvars[db_host]['ansible_host'] }}:{{ hostvars[db_host]['mysql_port'] }}
{% endfor %}
```

### inventory_hostname Magic Variable

The `inventory_hostname` contains the current host's name as defined in inventory.

```yaml
- hosts: all
  tasks:
    - name: Show hostname information
      debug:
        msg: |
          Inventory hostname: {{ inventory_hostname }}
          System hostname: {{ ansible_hostname }}
          FQDN: {{ ansible_fqdn }}
    
    - name: Host-specific configuration
      template:
        src: host_config.j2
        dest: "/etc/{{ inventory_hostname }}.conf"
    
    - name: Conditional task based on hostname
      service:
        name: nginx
        state: started
      when: inventory_hostname in groups['webservers']
```

### Cross-Host Variable Access Example

```yaml
---
- hosts: databases
  tasks:
    - name: Register database info
      set_fact:
        db_status: "active"
        db_connections: 50

- hosts: webservers  
  tasks:
    - name: Access database host variables
      debug:
        msg: |
          Database {{ item }} status: {{ hostvars[item]['db_status'] }}
          Connections: {{ hostvars[item]['db_connections'] }}
      loop: "{{ groups['databases'] }}"
    
    - name: Create config with all database IPs
      copy:
        content: |
          {% for db in groups['databases'] %}
          database_{{ loop.index }}={{ hostvars[db]['ansible_host'] }}
          {% endfor %}
        dest: /etc/app/databases.conf
```

## Ansible Facts

Ansible facts are system information automatically gathered from managed hosts. They provide details about hardware, operating system, network, and other host characteristics.

### Setup Module - Manual Fact Gathering

```yaml
- hosts: webservers
  gather_facts: false  # Disable automatic gathering
  tasks:
    - name: Manually gather facts
      setup:
    
    - name: Gather only network facts
      setup:
        filter: "ansible_default_ipv4"
    
    - name: Gather specific fact subsets
      setup:
        gather_subset:
          - "!all"
          - "network"
          - "hardware"
```

### Automatic Fact Gathering

```yaml
# Facts are gathered automatically by default
---
- hosts: webservers
  # gather_facts: true (default)
  tasks:
    - name: Use gathered facts
      debug:
        msg: |
          OS: {{ ansible_distribution }} {{ ansible_distribution_version }}
          Architecture: {{ ansible_architecture }}
          Memory: {{ ansible_memtotal_mb }}MB
          IP Address: {{ ansible_default_ipv4.address }}
```

### Debug ansible_facts Variable

```yaml
- hosts: webservers
  tasks:
    - name: Show all facts
      debug:
        var: ansible_facts
    
    - name: Show specific fact categories
      debug:
        msg: |
          System Facts:
          - Hostname: {{ ansible_facts['hostname'] }}
          - OS Family: {{ ansible_facts['os_family'] }}
          - Kernel: {{ ansible_facts['kernel'] }}
          
          Network Facts:
          - Default IPv4: {{ ansible_facts['default_ipv4']['address'] }}
          - All IPv4: {{ ansible_facts['all_ipv4_addresses'] }}
          
          Hardware Facts:
          - CPU Count: {{ ansible_facts['processor_vcpus'] }}
          - Memory: {{ ansible_facts['memtotal_mb'] }}MB
    
    - name: Filter facts by pattern
      debug:
        msg: "{{ item }}"
      loop: "{{ ansible_facts | dict2items | selectattr('key', 'match', 'ansible_eth.*') | list }}"
```

### Disabling Fact Gathering

```yaml
# Disable for entire playbook
---
- hosts: webservers
  gather_facts: false
  tasks:
    - name: Task without facts
      debug:
        msg: "No facts gathered - faster execution"

# Disable globally in ansible.cfg
# [defaults]
# gathering = explicit

# Disable for specific hosts in inventory
[webservers]
web1.example.com
web2.example.com gather_facts=false

# Conditional fact gathering
---
- hosts: all
  gather_facts: "{{ gather_host_facts | default(true) }}"
  tasks:
    - name: Run with: ansible-playbook -e "gather_host_facts=false" playbook.yml
      debug:
        msg: "Facts gathering controlled by variable"
```

### Custom Facts

```yaml
- hosts: webservers
  tasks:
    - name: Create custom fact directory
      file:
        path: /etc/ansible/facts.d
        state: directory
    
    - name: Add custom fact file
      copy:
        content: |
          {
            "app_version": "1.2.3",
            "environment": "production",
            "last_deployed": "2024-01-15"
          }
        dest: /etc/ansible/facts.d/custom.fact
    
    - name: Re-gather facts to include custom facts
      setup:
    
    - name: Use custom facts
      debug:
        msg: |
          Custom facts: {{ ansible_local.custom }}
          App version: {{ ansible_local.custom.app_version }}
```

### Fact Caching

```yaml
# ansible.cfg
[defaults]
fact_caching = jsonfile
fact_caching_connection = /tmp/ansible_facts_cache
fact_caching_timeout = 3600

# Facts will be cached and reused for 1 hour
---
- hosts: webservers
  tasks:
    - name: Facts loaded from cache if available
      debug:
        msg: "Using cached facts for faster execution"
```

