---
title: NFS最佳实践
date: 2024-07-11 20:30:04
slug: nfs-best-practices
tags:
  - k8s
  - Kubernetes
  - NFS
categories:
  - Kubernetes
  - CloudNative
---

本实践主要是在Ubuntu 24.04和kubernetes v1.29.6环境。内容主要包含两部分：在Ubuntu系统安装和配置NFS服务器和在Kubernetes集群配置NFS。

## 在Ubuntu上安装和配置NFS

### 服务器端

#### 安装NFS服务器内核
```shell
sudo apt-get update
sudo apt-get install nfs-kernel-server -y
```

#### 创建NFS导出(共享)目录
```shell
# 创建共享目录
sudo mkdir -p /mnt/nfs_share
# 允许所有客户端能访问共享目录
sudo chown -R nobody:nogroup /mnt/nfs_share/
# 给共享目录赋予权限
sudo chmod 777 /mnt/nfs_share/
```
#### 授予客户端共享目录访问权限
```shell
sudo vim /etc/exports

# 允许某个子网网段IP可以访问
/mnt/nfs_share 192.168.1.0/24(rw,sync,no_subtree_check)
```
- rw：代表读/写。
- sync：要求在应用更改之前将其写入磁盘。
- no_subtree_check：消除子树检查。

#### 导出NFS共享目录
```shell
sudo exports -a

# 重启NFS服务
sudo systemctl restart nfs-kernel-server
```

#### 允许某个子网网段IP通过防火墙访问NFS
```shell
sudo ufw allow from 192.168.1.0/24 to any port nfs
sudo ufw enable
sudo ufw status
```

###  客户端
####  安装客户端包
```shell
sudo apt update
sudo apt install nfs-common -y
```

#### 在客户端创建NFS挂载点
```shell
sudo mkdir -p /mnt/nfs_clientshare

# 将NFS服务器共享目录挂在到客户端
sudo mount <nfs_server_ip>:/mnt/nfs_share /mnt/nfs_clientshare
```

####  测试NFS共享
```shell
# 登录到NFS服务器
cd /mnt/nfs_share
touch test1.txt test2.txt test3.txt
# 登录到NFS客户端
cd /mnt/nfs_clientshare
ls -l
```

## 在Kubernetes中配置NFS
### 在k8s节点上安装NFS客户端包
```shell
sudo apt update
sudo apt install nfs-common -y
```

### 使用Helm安装和配置NFS Client Provisioner
```shell
helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/

helm install nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner \
    --set nfs.server=<nfs_server_ip> \
    --set nfs.path=/mnt/nfs_share \
	--set storageClass.onDelete=true
```

### 创建PVC Volume 测试
```yaml
# nfs-pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nfs-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: nfs-client
  resources:
    requests:
      storage: 5Gi
```

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
  name: nfs-nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      volumes:
        - name: nfs-nginx
          persistentVolumeClaim:
            claimName: nfs-pvc
      containers:
        - image: nginx
          name: nginx
          volumeMounts:
            - name: nfs-nginx
              mountPath: /usr/share/nginx/html
```

```shell
kubectl apply -f nfs-pvc.yaml
kubectl apply -f deployment.yaml
```

### 登录到NFS服务器查看
```shell
kubectl get po
kubectl exec -it nfs-nginx-xxx sh
# now we are into pod
# cd /usr/share/nginx/html
# ls -l
# echo "hello world" >index.html
# ls -l
# exit
```

```
cd /mnt/nfs_share
ls -l
```

## 引用文献
- [1] https://www.tecmint.com/install-nfs-server-on-ubuntu/
- [2] https://hbayraktar.medium.com/how-to-setup-dynamic-nfs-provisioning-in-a-kubernetes-cluster-cbf433b7de29
