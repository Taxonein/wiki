---
title: Ansible
description: 
published: true
date: 2024-09-16T12:41:24.436Z
tags: 
editor: markdown
dateCreated: 2024-09-16T12:41:24.436Z
---

# Ansible
Примеры файлов ansible:


`ansible.cfg:`
```yaml
[defaults]

inventory = ./ansible/hosts
host_key_checking = false
```

`hosts:`
```yaml
[test]
10.10.10.40
#client01
[clients]
client02 ansible_host=10.10.10.41 ansible_user=root ansible_ssh_private_key_file=/root/.ssh/sshkey
client03 ansible_host=10.10.10.42 ansible_user=root ansible_ssh_private_key_file=/root/.ssh/sshkey

[all_groups:children]
test
#clients
```

`test:`
```yaml
ansible_host: 10.10.10.40
ansible_user: root
ansible_ssh_private_key_file: /root/.ssh/sshkey
dev: taxonein
```



Примеры файлов опроса:

`ping.yml`
```yaml
- name: Ping Servers
  hosts: all_groups
  become: yes

  vars:
    packages:
              - nano
              - mc
              - htop


  tasks:
  - name: Task ping
    ping:
  - name: Upgrade system
    apt:
      upgrade: yes
#  - name: Install pkgs
#    apt:
#      pkg: "{{packages}}"
#      state: present
  - debug:
      msg: "{{ansible_distribution}} Version: {{ansible_distribution_version}}"
```