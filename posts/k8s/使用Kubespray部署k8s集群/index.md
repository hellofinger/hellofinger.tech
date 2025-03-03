---
title: 使用Kubespray部署Kubernetes集群
date: 2024-04-11 20:30:04
slug: install-kubernetes-using-kubespray
tags:
  - k8s
  - Kubernetes
categories:
  - Kubernetes
  - CloudNative
---

## 环境准备
一共4个节点，所有节点都使用ubuntu 24.04版本。
- Bastion (192.168.70.128)  堡垒机或者Ansbile操作节点，所有Kubernetes集群的操作都必须通过堡垒机。
- Controller (192.168.70.129) 控制器节点
- Node (192.168.70.130 & 192.168.70.131) 工作节点

## 下载Kubespray


将Kubespray代码克隆到本地，建议不要直接使用master分支，而是使用最近的release分支。
```
git clone https://github.com/kubernetes-sigs/kubespray.git
cd kubespray
git checkout release-2.25
```

```
ubuntu@bastion:~/kubespray$ git branch -a
  master
* release-2.25
  remotes/origin/HEAD -> origin/master
```

创建python虚拟环境，安装Ansible和其他依赖包
```
python3 -m venv venv
source venv/bin/activate

pip install -r requirements.txt
```

验证Ansible版本
```
(venv) ubuntu@bastion:~/kubespray$ ansible --version
ansible [core 2.16.9]
```

## 创建集群

### 修改配置

配置Bastion到集群节点免密登录
```
ssh-copy-id root@192.168.70.129
ssh-copy-id root@192.168.70.130
ssh-copy-id root@192.168.70.131
```

复制inventory/sample为inventory/mycluster
```
cp -rfp inventory/sample inventory/mycluster

declare -a IPS=(192.168.70.129 192.168.70.130 192.168.70.131)
CONFIG_FILE=inventory/mycluster/hosts.yaml python3 contrib/inventory_builder/inventory.py ${IPS[@]}
```

修改inventory/mycluster/hosts.yaml (根据自己的主机名修改节点名称)
```
all:
  hosts:
    controller:
      ansible_host: 192.168.70.129
      ip: 192.168.70.129
      access_ip: 192.168.70.129
    node1:
      ansible_host: 192.168.70.130
      ip: 192.168.70.130
      access_ip: 192.168.70.130
    node2:
      ansible_host: 192.168.70.131
      ip: 192.168.70.131
      access_ip: 192.168.70.131
  children:
    kube_control_plane:
      hosts:
        controller:
    kube_node:
      hosts:
        node1:
        node2:
    etcd:
      hosts:
        controller:
        node1:
        node2:
    k8s_cluster:
      children:
        kube_control_plane:
        kube_node:
    calico_rr:
      hosts: {}
```

国内部署，需要修改k8s镜像源 (https://github.com/kubernetes-sigs/kubespray/blob/master/docs/operations/mirror.md)
```
sed -i -E '/# .*\{\{ files_repo/s/^# //g' inventory/mycluster/group_vars/all/mirror.yml
tee -a inventory/mycluster/group_vars/all/mirror.yml <<EOF
gcr_image_repo: "gcr.m.daocloud.io"
kube_image_repo: "k8s.m.daocloud.io"
docker_image_repo: "docker.m.daocloud.io"
quay_image_repo: "quay.m.daocloud.io"
github_image_repo: "ghcr.m.daocloud.io"
files_repo: "https://files.m.daocloud.io"
EOF
```

检查参数配置
```
cat inventory/mycluster/group_vars/all/all.yml
cat inventory/mycluster/group_vars/k8s_cluster/k8s-cluster.yml
```

### 安装集群
```
ansible-playbook -i inventory/mycluster/hosts.yaml  --become --become-user=root cluster.yml
```

### 验证集群
登录到Controller节点，或者将Controller节点上的kube config文件复制到Bastion节点.
```
root@controller:~/.kube# kubectl get nodes
NAME         STATUS   ROLES           AGE    VERSION
controller   Ready    control-plane   218d   v1.29.6
node1        Ready    <none>          218d   v1.29.6
node2        Ready    <none>          218d   v1.29.6
```

