# Ansible playbooks

[Source](https://docs.ansible.com/ansible/latest/user_guide/playbooks_intro.html)

Sample playbook (`debian-curl.yaml`)

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