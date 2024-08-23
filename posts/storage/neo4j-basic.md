---
title: Neo4j 基础知识
date: 2020-06-16T15:02:52+08:00
slug: neo4j-basic
tags:
  - Neo4j
categories:
  - Database
  - Storage
---

## 简介

图形数据库： 存储 查询/遍历 多个连接数据

Neo4j是一个嵌入式的、基于磁盘的、具备完全的事务特性的Java持久化引擎，但是它将结构化数据存储在网络(从数学角度叫做图)上而不是表中。Neo4j也可以被看作是一个高性能的图引擎，该引擎具有成熟数据库的所有特性。

节点/关系 用自增id来唯一标识， 容量35亿； 关系应该是方向性的。

| 关系数据库 | 图形数据库   |
| ---------- | ------------ |
| 表         | 图           |
| 行         | 节点         |
| 列和数据   | 属性和值(KV) |
| Join       | 边           |

## 下载

https://neo4j.com/download-center/#community

### Python 驱动

https://github.com/neo4j/neo4j-python-driver

## Cypher Query Language (CQL)

https://neo4j.com/developer/cypher-query-language/

### 创建节点

语法：CREATE (节点名: 标签 {节点属性})

```cypher
# 一个标签的节点
CREATE (node:label)

# 多个标签的节点
CREATE (node:label_1:label_2:label_3...label_N)

# 创建节点属性
CREATE (node:label { property_1: value_1, property_2: value_2...})

CREATE (p:Person {name: "Finger", city: "HengYang"}) RETURN p

```

### MERGE

合并节点或者关系，属于先读后写操作，相当于MATCH + CREATE，先检查数据库中节点/关系是否存在，如果存在的话就不再创建，反之执行CREATE。

```cypher
// 给节点a, b建立关系，如果a,b已经存在，就无需新建。 
MATCH (a:Person {name:"Finger"}), (b:Person{name:"Jack Ma"}) MERGE (a)-[:KNOWS]->(b)
```

```json
[
    {
    "id":1, "friends":[2,3], "name": "Bob", "age": 27,
    "book":[{"name":"book1", "year":2000}, {"name":"book3", "year":1990}]
    },
    {
    "id":2, "friends":[1], "name": "Alice", "age": 29,
    "book":[{"name":"book1", "year":2000}, {"name":"book2", "year":1999}]
    },
    {
    "id":3, "friends":[2], "name": "John", "age": 20,
    "book":[{"name":"book3", "year":1990}]
    }
]
```

### 删除节点

语法：MATCH (节点名：标签) DELETE 节点名

```cypher
MATCH (p:Persion) WHERE p.name = "Finger" DELETE p
```

### 查找节点

语法：MATCH (变量名：匹配的标签) WHERE 过滤结果 RETURN 返回特定结果

```cypher
MATCH (p:Persion) WHERE p.name = "Finger" RETURN p.name
```

### 更新节点

语法：MATCH (变量名: 匹配的标签) WHERE 过滤结果 SET 变量名.属性名=值 RETURN 变量名

```cypher
MATCH (p:Persion) WHERE p.name = "Finger" SET p.city = "ChangSha" RETURN p
```

### LIMIT

```cypher
MATCH (p:Persion) RETURN p LIMIT 10
```

## 参考

- http://neo4j.com.cn/public/docs/index.html

<!--more-->
