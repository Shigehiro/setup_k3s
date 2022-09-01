# Set up k3s
---

## Description

This playbook deploys one k3s master node and multiple k3s worker nodes.
I have tested this playbook on AlmaLinux 8.6.

## Ansible vars

```text
$ cat group_vars/k3s.yml 
package_list:
  - bash-completion
  - tree
master_ip: 192.168.124.155
```

## Environment Setup

### prepare at least two RedHat8 based nodes. (RHEL8, AlmaLinux8, RockyLinux8)

- one master node and multiple worker nodes.
- note that this playbook does not support deploying multiple master nodes.

### python3 setup

```

```

### edit Ansible inventory file and group vars to meet your environment

- inventory.ini
- group_vars/k3s.yml

### run the playbook


## How to uninstall k3s on all nodes

```
ansible-playbook -i ./inventory.ini uninstall_k3s.yml
```