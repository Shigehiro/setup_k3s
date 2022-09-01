# Set up k3s

- [Set up k3s](#set-up-k3s)
  - [Description](#description)
  - [Ansible vars](#ansible-vars)
  - [Environment Setup](#environment-setup)
    - [prepare at least two RedHat8 based nodes. (RHEL8, AlmaLinux8, RockyLinux8)](#prepare-at-least-two-redhat8-based-nodes-rhel8-almalinux8-rockylinux8)
    - [python3 setup](#python3-setup)
    - [edit Ansible inventory file and group vars to meet your environment](#edit-ansible-inventory-file-and-group-vars-to-meet-your-environment)
    - [run the playbook](#run-the-playbook)
  - [How to uninstall k3s on all nodes](#how-to-uninstall-k3s-on-all-nodes)

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

```text
# cat /etc/almalinux-release
AlmaLinux release 8.6 (Sky Tiger)

# python3 -m venv ansible_env
# source ansible_env/bin/activate
# pip3 install --upgrade pip
# pip3 install -r requirements.txt 
```

### edit Ansible inventory file and group vars to meet your environment

- inventory.ini
- group_vars/k3s.yml

### run the playbook


```
# grep -E "^[a-zA-Z]|\[.*\]" inventory.ini 
[master]
lab02-m01 ansible_connection=local
[worker]
lab02-w01 ansible_host=192.168.124.156
lab02-w02 ansible_host=192.168.124.157
[k3s:children]
master
worker
[k3s:vars]
ansible_user=root
```

```text
# cat group_vars/k3s.yml 
package_list:
  - bash-completion
  - tree
master_ip: 192.168.124.155
```

```text
# ansible-playbook -i ./inventory.ini main.yml 

PLAY RECAP ************************************************************************************************************************************************************************
lab02-m01                  : ok=20   changed=4    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0   
lab02-w01                  : ok=9    changed=2    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0   
lab02-w02                  : ok=9    changed=2    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0   
```

on the master node.
```text
# kubectl cluster-info 
Kubernetes control plane is running at https://127.0.0.1:6443
CoreDNS is running at https://127.0.0.1:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
Metrics-server is running at https://127.0.0.1:6443/api/v1/namespaces/kube-system/services/https:metrics-server:https/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.

# kubectl get nodes
NAME        STATUS   ROLES                  AGE    VERSION
lab02-m01   Ready    control-plane,master   4m9s   v1.24.4+k3s1
lab02-w01   Ready    <none>                 94s    v1.24.4+k3s1
lab02-w02   Ready    <none>                 90s    v1.24.4+k3s1
#

# kubectl get pod -n kube-system 
NAME                                      READY   STATUS      RESTARTS   AGE
local-path-provisioner-7b7dc8d6f5-8x5dr   1/1     Running     0          12m
coredns-b96499967-k5f5f                   1/1     Running     0          12m
helm-install-traefik-crd-5sv5k            0/1     Completed   0          12m
metrics-server-668d979685-bsqk9           1/1     Running     0          12m
helm-install-traefik-48xzt                0/1     Completed   1          12m
svclb-traefik-70015156-dkvlc              2/2     Running     0          10m
traefik-7cd4fcff68-ks5sc                  1/1     Running     0          10m
svclb-traefik-70015156-8cn24              2/2     Running     0          9m26s
svclb-traefik-70015156-2992r              2/2     Running     0          9m21s
#
```

```text
# curl -k https://raw.githubusercontent.com/kubernetes/website/main/content/en/examples/controllers/nginx-deployment.yaml | kubectl apply -f -

# kubectl get pod -o wide 
NAME                                READY   STATUS    RESTARTS   AGE   IP          NODE        NOMINATED NODE   READINESS GATES
nginx-deployment-6595874d85-b4js7   1/1     Running   0          69s   10.42.3.3   lab02-w02   <none>           <none>
nginx-deployment-6595874d85-9tf2q   1/1     Running   0          69s   10.42.0.9   lab02-m01   <none>           <none>
nginx-deployment-6595874d85-wqd97   1/1     Running   0          69s   10.42.1.3   lab02-w01   <none>           <none>
# 

# kubectl get deployments.apps 
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           85s

# kubectl expose deployment nginx-deployment 
service/nginx-deployment exposed
# 

# kubectl get service|grep nginx
nginx-deployment   ClusterIP   10.43.91.164   <none>        80/TCP    2m57s

# curl -I 10.43.91.164
HTTP/1.1 200 OK
Server: nginx/1.14.2
Date: Thu, 01 Sep 2022 14:37:02 GMT
Content-Type: text/html
Content-Length: 612
Last-Modified: Tue, 04 Dec 2018 14:44:49 GMT
Connection: keep-alive
ETag: "5c0692e1-264"
Accept-Ranges: bytes

# kubectl delete service nginx-deployment 
service "nginx-deployment" deleted

# kubectl delete deployments.apps nginx-deployment 
deployment.apps "nginx-deployment" deleted
# 
```

## How to uninstall k3s on all nodes

```
ansible-playbook -i ./inventory.ini uninstall_k3s.yml
```