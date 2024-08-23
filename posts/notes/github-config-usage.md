---
title: github 本地配置
tags:
  - Git
  - Github
categories:
  - Note
author: Finger
date: 2017-01-13 17:56:00
---


### 本地配置github用户名密码

#### 1、使用命令
```
git config --global user.name [username]
git config --global user.email [email]
git config --global credential.helper store
```

#### 2、配置文件
```
[user]
	name = finger
	email = finger_chou@163.com
```

### 保存用户名密码

#### 1、使用命令
```
echo "[credential]" >> .git/config
echo "    helper = store" >> .git/config
```

#### 2、配置文件
```
[credential]
    helper = store
```

### 查看配置
```
git config --list
```

