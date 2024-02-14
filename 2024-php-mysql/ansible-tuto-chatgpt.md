Certainly! Let's start with a concise Ansible tutorial that covers installing PHP and MySQL on a target host using Alpine or Debian Linux distributions. We'll also look into structuring this setup in an Ansible playbook, sharing the playbook, and managing secrets securely.

### 1. Installing PHP + MySQL on a Target Host

For this example, let's assume we're targeting a Debian-based system for simplicity. The approach for Alpine would be similar, but the package names and the package manager (`apk` instead of `apt`) would differ.

First, you need to ensure Ansible is installed on your control machine (the machine from which you are running Ansible commands). You can install Ansible on most Linux distributions using the default package manager.

#### Ansible Playbook Structure

An Ansible playbook is a YAML file that describes the desired state of your target machines. Here is a basic example of a playbook (`install_php_mysql.yml`) that installs PHP and MySQL on a Debian-based system:

```yaml
---
- name: Install PHP and MySQL on Debian-based systems
  hosts: all
  become: yes  # Use privilege escalation to become the root user
  tasks:
    - name: Update apt cache
      ansible.builtin.apt:
        update_cache: yes
        cache_valid_time: 3600  # Update the cache if it's older than an hour

    - name: Install PHP
      ansible.builtin.apt:
        name: php
        state: present

    - name: Install MySQL Server
      ansible.builtin.apt:
        name: mysql-server
        state: present

    - name: Start MySQL Service
      ansible.builtin.service:
        name: mysql
        state: started
        enabled: yes  # Ensure the MySQL service starts on boot
```

### 2. Structuring Your Playbook

Your Ansible project can be structured as follows:

```
ansible-project/
│
├── inventory/
│   └── hosts  # Your inventory file where you define your target machines
│
├── group_vars/
│   └── all.yml  # Variables that apply to all hosts
│
└── playbooks/
    └── install_php_mysql.yml  # Your playbook file
```

- **Inventory (`hosts`)**: Defines groups of hosts. An example entry for a Debian host would be:
  ```
  [debian_servers]
  your_debian_server ansible_host=example.com
  ```

- **Group Vars (`all.yml`)**: Here, you can define variables that are common across all hosts. For example, you could define the PHP and MySQL versions you want to install.

- **Playbook (`install_php_mysql.yml`)**: Contains the tasks to be performed, as shown above.

### 3. Sharing Your Playbook

To share your playbook, you can use a version control system like Git. Push your Ansible project to a repository on GitHub, GitLab, or Bitbucket. This makes it easy to version your infrastructure as code and collaborate with others.

### 4. Storing Secrets

For storing secrets like database passwords, Ansible provides a feature called Ansible Vault. With Ansible Vault, you can encrypt sensitive data, making it safe to store in version control.

- **Creating an Encrypted File**: Use the `ansible-vault create <file_name>` command. For example, `ansible-vault create secrets.yml`.
- **Editing an Encrypted File**: Use the `ansible-vault edit secrets.yml`.
- **Using Encrypted Variables in Playbooks**: Reference them just like any other variable. Ansible decrypts them on the fly when running the playbook.

When you run a playbook that uses Vault-encrypted files, you'll need to provide the Vault password. Use the `--ask-vault-pass` flag or store the password in a file and use the `--vault-password-file <file_name>` option.

This tutorial gives you a basic framework to start with Ansible for deploying PHP and MySQL. You can expand on this by adding more roles, tasks, and optimizing your playbook structure as your infrastructure grows. If you have more specific needs or run into any issues, feel free to ask!
