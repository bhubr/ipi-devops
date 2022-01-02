# Ansible sample playbooks

[Source](https://docs.ansible.com/ansible/latest/user_guide/playbooks_intro.html)

## 1. Verify that `curl` is installed (`debian-curl.yaml`)

```yaml
---
- name: Install curl
  hosts: vms
  remote_user: root

  tasks:
  - name: Ensure curl is at the latest version
    ansible.builtin.apt:
      name: curl
      state: latest
```

## 2. Install and deploy MEAN stack (`debian-mean.yaml`)

Overview

* Install Nginx
* **Install GnuPG**! Otherwise MySQL APT repo doesn't install
* Install MySQL (or MongoDB)

    * Download APT repository: <https://dev.mysql.com/get/mysql-apt-config_0.8.20-1_all.deb>
    * Use [get_url](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/get_url_module.html) for that&hellip; It even has a "Download file with check (md5)" example!
    * Ideally check md5sum: `799bb0aefb93d30564fa47fc5d089aeb`
* Install Node.js
* Install pm2
* Clone repos (front and back)
* Build front app

```yaml
---
- name: Install curl
  hosts: vms
  remote_user: root

  tasks:
  - name: Ensure curl is at the latest version
    ansible.builtin.apt:
      name: curl
      state: latest
```

debconf-set-selections <<< 'mysql-apt-config mysql-apt-config/repo-codename select buster'
debconf-set-selections <<< 'mysql-apt-config mysql-apt-config/repo-distro select debian'
debconf-set-selections <<< 'mysql-apt-config mysql-apt-config/repo-url string http://repo.mysql.com/apt'
debconf-set-selections <<< 'mysql-apt-config mysql-apt-config/select-preview select '
debconf-set-selections <<< 'mysql-apt-config mysql-apt-config/select-product select Ok'
debconf-set-selections <<< 'mysql-apt-config mysql-apt-config/select-server select mysql-5.7'
debconf-set-selections <<< 'mysql-apt-config mysql-apt-config/select-tools select '
debconf-set-selections <<< 'mysql-apt-config mysql-apt-config/unsupported-platform select abort'
