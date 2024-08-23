---
title: Ubuntu 安装svn服务
author: Finger
tags:
  - SVN
  - Ubuntu
categories:
  - Note
date: 2017-01-27 20:56:00
---

### 一、安装Subversion Server

```
apt-get install subversion
```

### 二、创建SVN版本库
```
mkdir /srv/svn          # 创建SVN根目录
svnadmin create test    # 创建测试项目库
```

### 三、SVN配置
主要包含以下三个配置文件：

##### 1. svnserve.conf    

服务配置：
```
[general]
anon-access = none      # 匿名用户不可读
auth-access = write     # 权限用户可写
password-db = passwd    # 启用密码文件
authz-db = authz        # 启用权限文件
realm = repos           # 认证域名称
```
##### 2. passwd           

账号配置：
```
[user]
finger = 123456
```
##### 3. authz            

权限配置：
```
[groups]
admin = finger

[/]
@admin = rw         # admin组拥有所有读写权限
* = r               # 其他只有读权限
```

### 四、启动和停止

```
启动：
svnserve -d -r /srv/svn   # -d 表示后台运行 -r 表示根目录

停止：
ps -ef | grep svn   # 查看svn进程ID

kill -9 进程ID
```

### 五、测试

```
svn co svn://127.0.0.1/test
```

### 六、问题
**1、svn: E220003: Invalid authz configuration**

原因是authz文件配置错误，仔细检查authz文件。


### 七、参考文献
https://my.oschina.net/jast90/blog/382688

http://www.linuxidc.com/Linux/2015-01/111956.htm