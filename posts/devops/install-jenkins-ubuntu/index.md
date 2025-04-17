---
title: 在Ubuntu安装Jenkins
date: 2024-04-12 21:30:04
slug: install-jenkins-ubuntu
tags:
  - Jenkins
  - DevOps
categories:
  - DevOps
---

测试环境：Ubuntu 24.04

## 安装Java运行时

```
sudo apt-get update
sudo apt install openjdk-21-jdk -y
java -version
```

![](./imgs/java-version.png)

## 安装Jenkins

### 添加Jenkins Repository
```shell
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
  
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
```

### 安装Jenkins
```shell
sudo apt-get update
sudo apt-get install jenkins
```

### 启动Jenkins服务
```shell
# 启动jenkins服务
sudo systemctl start jenkins

# 查看服务状态
sudo systemctl status jenkins
```

![](./imgs/jenkins-status.png)

### 解锁Jenkins站点

打开浏览器输入http://<ip_address>:8080，第一次打开会出现解锁Jenkins页面。

![](./imgs/unlock-jenkins.png)

```shell
# 按照页面提示，查看初始化密码
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```
![](./imgs/jenkins-init-password.png)

输入初始化密码后，进入到插件选择页面。

![](./imgs/jenkins-plugins-install.png)

### 创建管理员账号

![](./imgs/create-first-admin-user.png)

### 登录Jenkins站点

![](./imgs/jenkins-ready.png)

![](./imgs/welcome-jenkins.png)

## 添加节点

### 创建Credentials
![](./imgs/create-credentials.png)

### 添加节点
![](./imgs/new-node.png)

![](./imgs/node-setting1.png)

![](./imgs/node-setting2.png)

![](./imgs/node-list.png)

### 测试节点

创建一个Hello World Pipeline.
```shell
pipeline {
    agent {
        node {
            label 'docker'
        }
    }

    stages {
        stage('Hello') {
            steps {
                echo 'Hello World'
            }
        }
    }
}

```

![](./imgs/pipeline.png)

![](./imgs/pipeline-output.png)