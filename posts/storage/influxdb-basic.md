---
title: InfluxDB 基础知识
date: 2019-11-15T11:51:52+08:00
slug: influxdb-basic
tags:
  - InfluxDB
categories:
  - Database
  - Storage
---

## 简介

InfluxDB 是一个开源的时间序列数据库，专门设计用于处理时间相关数据。它被广泛用于监控、分析和存储大规模的时序数据，如应用程序指标、传感器数据、日志数据等。InfluxDB 使用一种称为 InfluxQL 的SQL-like 查询语言来查询和操作数据。它具有高性能、可扩展性和易用性的特点，适用于需要高速写入和查询时间序列数据的场景。

InfluxDB 提供了多种客户端库和集成，使其可以轻松与各种编程语言和数据处理工具集成。此外，InfluxDB 还支持数据的持续查询和连续写入，可以实时监控和分析数据。由于其设计的时序数据库特性，InfluxDB 在处理时间序列数据方面表现出色，并被广泛应用于监控系统、IoT（物联网）应用、日志分析等领域。


## 安装
```
# Ubuntu & Debian
wget https://dl.influxdata.com/influxdb/releases/influxdb_1.7.9_amd64.deb
sudo dpkg -i influxdb_1.7.9_amd64.deb

# or

wget -qO- https://repos.influxdata.com/influxdb.key | sudo apt-key add -
source /etc/lsb-release
echo "deb https://repos.influxdata.com/${DISTRIB_ID,,} ${DISTRIB_CODENAME} stable" | sudo tee /etc/apt/

sudo apt-get update && sudo apt-get install influxdb
sudo service influxdb start
```

## 卸载

```bash
# Ubuntu & Debian
sudo dpkg -l | grep influxdb
sudo apt-get autoremove influxdb

```

## 认证管理

### 修改配置文件

```bash
# /etc/influxdb/influxdb.conf

[http]
  enabled = true
  bind-address = ":8086"
  auth-enabled = true
  log-enabled = true
  write-tracing = false
  pprof-enabled = true
  pprof-auth-enabled = true
  debug-pprof-enabled = false
  ping-auth-enabled = true
```

```bash
# 重启
sudo systemctl restart influxdb 
# or
sudo service influxdb restart

# 客户端认证登录
influx -username devops -password xxx

# 测试
curl -G http://localhost:8086/query -u username:password --data-urlencode "q=SHOW DATABASES"
```

## 与关系型数据库比较

| Relational database | Influx DB   |
| ------------------- | ----------- |
| database            | database    |
| table               | measurement |
| column              | points      |

## 特殊名词

 Point由时间戳（time）、数据（field）、标签（tags）组成。 

| Point属性 | 传统数据库中的概念                                      |
| --------- | ------------------------------------------------------- |
| time      | 每个数据记录时间，是数据库中的主索引(会自动生成)        |
| fields    | 各种记录值（没有索引的属性）也就是记录的值：温度， 湿度 |
| tags      | 各种有索引的属性：地区，海拔                            |

## 用户管理

### 创建管理员账号

```sql
// 新用户
CREATE USER <username> WITH PASSWORD '<password>' WITH ALL PRIVILEGES

// 老用户
GRANT ALL PRIVILEGES TO <username>
```

### 创建普通用户

```sql
CREATE USER <username> WITH PASSWORD '<password>'

// 授权数据库权限
GRANT [READ,WRITE,ALL] ON <database_name> TO <username>

GRANT READ ON test TO "grafana"

GRANT ALL ON test TO "devops"
```

## 数据保存策略（Retention Policies）

```sql

# 创建策略
CREATE RETENTION POLICY aws_elb_log_retention_policy ON DevOpsBigScreen DURATION 3d REPLICATION 3 DEFAULT

# 查看策略
SHOW RETENTION POLICIES ON <database_name>

# 删除策略
DROP RETENTION POLICY <retention_policy_name> ON <database_name>

```

## 参考

- https://docs.influxdata.com/influxdb/v1.7/introduction/installation/
- https://www.influxdata.com/blog/getting-started-python-influxdb/
- https://xtutu.gitbooks.io/influxdb-handbook/content/shu_ju_ku_yu_biao_de_cao_zuo.html

<!--more-->