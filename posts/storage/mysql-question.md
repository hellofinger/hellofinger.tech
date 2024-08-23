---
title: Mysql遇到的问题笔记
tags:
  - Mysql
  - Database
categories:
  - Stroage
  - Database
author: Finger
date: 2017-01-03 10:43:00
---

问题描述：mysql远程连接错误10038--navicat for mysql (10038)

解决方案：
1、授权
```
mysql>grant all privileges on *.*  to  'root'@'%'  identified by 'youpassword'  with grant option;

mysql>flush privileges;
```

2、修改 /etc/mysql/mysql.conf.d
```
找到bind-address = 127.0.0.1这一行

改为bind-address = 0.0.0.0即可

```

如果还没解决可能是远程端口（3306）未对外开放。
需要修改安全组的入站规则。 

---

问题描述:
ERROR 1030 (HY000): Got error 28 from storage engine.

解决办法:
查了一下，数据库文件所在的盘应该没事，应该是数据库用的临时目录空间不够。或者将tmpdir指向一个硬盘空间很大的目录即可。

---

问题描述:
Can't open the mysql.plugin table. Please run mysql_upgrade to create it.

解决办法：
```
sudo vim /etc/apparmor.d/usr.sbin.mysqld

added:
/var/mysql r,
/var/mysql** rwk,

sudo /etc/init.d/apparmor restart
sudo service mysql start
```

---

问题描述:
marked as crashed and last (automatic?) repair failed

解决办法:
```
sudo service mysql stop
cd /dadadir/mysql/database
sudo myisamchk -r *.MYI
```

PS: 迁移数据前，一定要stop mysql不然就会出现上诉问题


---

导出mysql查询结果
select date_time, uid from register where package = 'mumayi' into outfile "/tmp/uid.txt";
导出查询结果：Select语句 into outfile '保存路径+文件名';
导入查询结果：load data local infile '保存路径+文件名' into table 表明 character set utf8;