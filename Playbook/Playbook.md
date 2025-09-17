# Ansible Playbook
An Ansible Playbook is a YAML file that defines a series of tasks to be executed on managed nodes. It is where we define what we want Ansible to do.

- **Playbook** - A single YAML file
  - **Play** - Defines a set of activities (tasks) to be run on hosts
    - **Task** - An action to be performed on the host based from its order
      - Execute a command
      - Run a script
      - Install a package
      - Shutdown/Restart 

## Structure of a Playbook
A playbook consists of one or more "plays". Each play maps a group of hosts to tasks.
```yaml
- name: Play 1
  hosts: webservers
  become: yes
  tasks:
    - name: Install Nginx
      apt:
        name: nginx
        state: present
- name: Play 2
  hosts: webservers
  become: yes
  tasks:
    - name: Start web server
      service:
        name: nginx
        state: present
```

### Host
The `hosts` directive specifies the group of hosts or individual host on which the tasks will be executed. This can refer to groups defined in your Ansible ***inventory file***.

The host defined in an inventory file must match the host used in the Playbooks, and all connection information for the host is retrieved from the inventory file.

### Module
The different actions run by task are called modules. For instance, 

- **command**
- **script**
- **yum**
- **service**

#### Basic module command
```yaml
- name: Install Nginx
  apt:
    name: nginx
    state: present
```

### Run
To execute a playbook, use the `ansible-playbook` command followed by the playbook file name:
```bash
ansible-playbook playbook.yml
```
#### Get to know more about Run
To get more information about the `ansible-playbook` command and its options, you can use:
```bash
ansible-playbook --help
```

## Verifying Playbooks
Running critical software update accross hundreds of servers might need verification. Unnoticed errors in the playbook might shutdown all service in each servers. Verifying a playbook before executing it in a production environment is a crucial practice to catch all the errors and avoid the risk of down time, data loss or other issues.

### Check Mode
Check mode is a dry-run mode where Ansible executes the playbook without making any actual changes on the host.

- Ansible's "dry-run" where no actual changes are made on hosts
- Allow preview of playbook changes without applying them
- Use the `--check` option to run a playbook in check mode

**Please note that not all Anisble modules support check mode. If check mode is not supported, the task will be skipped.**

```bash
ansible-playbook install_nginx.yml --check
```

*install_nginx.yml*
```yaml
- hosts: webservers
  task:
    - name: Ensure nginx is installed
      apt:
        name: nginx
        state: present
      become: yes
```
### Diff Mode
Shows the differences between the current state and the state after the playbook is run. 

- Provides a before-and-after comparison of playbook changes
- Understand and verify the impact of playbook changes before applying them
- Utilize the `--diff` option to run a playbook in diff mode

```bash
ansible-playbook configure_nginx.yml --diff
```

*configure_nginx.yml*
```yaml
- hosts: webservers
  tasks:
    - name: Update Nginx configuration
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
      notify:
        - Restart Nginx
      become: yes

  handlers:
    - name: Restart Nginx
      service:
        name: nginx
        state: restarted
      become: yes
```
### Check and Diff mode
You can combine both check and diff modes to preview changes and see the differences without applying them.
```bash
ansible-playbook playbook.yml --check --diff
```

### Symtax Check
Before running a playbook, it's a good practice to check its syntax to catch any YAML formatting errors or Ansible-specific issues.
```bash
ansible-playbook playbook.yml --syntax-check
```

## Ansible-lint
Ansible-lint is a tool that checks Ansible playbooks for best practices and common mistakes. It helps ensure that your playbooks are well-structured and follow recommended guidelines.
```bash
ansible-lint playbook.yml
```
