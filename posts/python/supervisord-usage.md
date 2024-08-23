---
title: Supervisord 使用笔记
tags:
  - Supervisord
categories: 
  - Python
author: Finger
date: 2017-06-22 23:40:00
---


#### 一、安装

```
- pip install supervisor  
- echo_supervisord_conf  #或者  
- echo_supervisord_conf > supervisord.conf  
- supervisord -c ./supervisord.conf  
```

#### 二、遇到错误

```
[root@localhost ~]# supervisord -c ./supervisord.conf   
Error: .ini file does not include supervisord section  
For help, use /usr/bin/supervisord -h  
```

**错误原因是因为配置文件少了 [supervisrod] 配置项**
```
[program:djangotest]  
user=lzz  
command=/usr/bin/python /var/www/djangotest/manage.py runserver 0.0.0.0:8000  
autostart=true  
autorestart=true  
stderr_logfile=/var/logs/err.log  
stdout_logfile=/var/logs/out.log  
stopsignal=INT  
   
[supervisord]  
```

#### 三、常用命令
```
supervisord : supervisor的服务器端部分，用于supervisor启动
supervisorctl：启动supervisor的命令行窗口，在该命令行中可执行start、stop、status、reload等操作。
```

#### 参考文献

http://www.jianshu.com/p/9abffc905645