---
title: Python Elasticsearch DSL 使用笔记(一)
tags:
  - Python
  - ElasticSearch
  - SearchEngine
categories:
  - ElasticSearch
  - Storage
author: Finger
date: 2017-08-12 20:39:00
---


**下载地址**: https://www.elastic.co/downloads/elasticsearch

**使用版本**: elasticsearch-5.5.1

**Python Elasticsearch DSL**: https://github.com/elastic/elasticsearch-dsl-py

**测试代码**： https://github.com/zhoujun/mydemos/tree/master/elasticsearch-demo

### 一、Elasticsearch的基本概念
- Index：Elasticsearch用来存储数据的逻辑区域，它类似于关系型数据库中的database 概念。一个index可以在一个或者多个shard上面，同时一个shard也可能会有多个replicas。
- Document：Elasticsearch里面存储的实体数据，类似于关系数据中一个table里面的一行数据。 document由多个field组成，不同的document里面同名的field一定具有相同的类型。document里面field可以重复出现，也就是一个field会有多个值，即multivalued。
- Document type：为了查询需要，一个index可能会有多种document，也就是document type. 它类似于关系型数据库中的 table 概念。但需要注意，不同document里面同名的field一定要是相同类型的。
- Mapping：它类似于关系型数据库中的 schema 定义概念。存储field的相关映射信息，不同document type会有不同的mapping。

下图是ElasticSearch和关系型数据库的一些术语比较：

| Relationnal database        	| Elasticsearch           
| ------------- |:-------------
| Database      | Index 		 
| Table      	| Type      	   
| Row 			| Document          
| Column 		| Field      	  
| Schema 		| Mapping        
| Index 		| Everything is indexed       
| SQL 			| Query DSL       
| SELECT * FROM table... 			| GET http://...  
| UPDATE table SET 			| PUT http://...   

### 二、Python Elasticsearch DSL使用简介

#### 1、安装
```unix
$ pip install elasticsearch-dsl
```

#### 2、创建索引和文档
```python
from datetime import datetime
from elasticsearch_dsl import DocType, Date, Integer, Keyword, Text
from elasticsearch_dsl.connections import connections

# Define a default Elasticsearch client
connections.create_connection(hosts=['localhost'])

class Article(DocType):
    title = Text(analyzer='snowball', fields={'raw': Keyword()})
    body = Text(analyzer='snowball')
    tags = Keyword()
    published_from = Date()
    lines = Integer()

    class Meta:
        index = 'blog'

    def save(self, ** kwargs):
        self.lines = len(self.body.split())
        return super(Article, self).save(** kwargs)

    def is_published(self):
        return datetime.now() >= self.published_from

# create the mappings in elasticsearch
Article.init()

```

创建了一个索引为blog，文档为article的Elasticsearch数据库和表。
必须执行`Article.init()`方法。 这样Elasticsearch才会根据你的DocType产生对应的Mapping。否则Elasticsearch就会在你第一次创建Index和Type的时候根据你的内容建立对应的Mapping。


现在我们可以通过Elasticsearch Restful API来检查
```
http GET http://127.0.0.1:9200/blog/_mapping/

{"blog":
	{"mappings":
		{"article":
			{"properties":{
				"body":{"type":"text","analyzer":"snowball"},
				"lines":{"type":"integer"},
				"published_from":{"type":"date"},
				"tags":{"type":"keyword"},
				"title":{"type":"text","fields":{"raw":{"type":"keyword"}},"analyzer":"snowball"}
			}
		}}
	}
}

```
### 三、使用Elasticsearch进行CRUD操作

#### 1、Create an article
```python
# create and save and article
article = Article(meta={'id': 1}, title='Hello elasticsearch!', tags=['elasticsearch'])
article.body = ''' looong text '''
article.published_from = datetime.now()
article.save()
```

=>Restful API
```
http POST http://127.0.0.1:9200/blog/article/1 title="hello elasticsearch" tags:='["elasticsearch"]'

HTTP/1.1 201 Created
Content-Length: 73
Content-Type: application/json; charset=UTF-8

{
    "_id": "1", 
    "_index": "blog", 
    "_type": "article", 
    "_version": 1, 
    "created": true
}
```

#### 2、Get a article
```python
article = Article.get(id=1)

# 如果获取一个不存在的文章则返回None
a = Article.get(id='no-in-es')
a is None

# 还可以获取多个文章
articles = Article.mget([1, 2, 3])
```

=>Restful API
```
http GET http://127.0.0.1:9200/blog/article/1

HTTP/1.1 200 OK
Content-Length: 141
Content-Type: application/json; charset=UTF-8

{
    "_id": "1", 
    "_index": "blog", 
    "_source": {
        "tags": [
            "elasticsearch"
        ], 
        "title": "hello elasticsearch"
    }, 
    "_type": "article", 
    "_version": 1, 
    "found": true
}
```

#### 3、Update a article
```python
article = Article.get(id=1)
article.tags = ['elasticsearch', 'hello']
article.save()

# 或者
article.update(body='Today is good day!', published_by='me')
```

=>Restful API
```
http PUT http://127.0.0.1:9200/blog/article/1 title="hello elasticsearch" tags:='["elasticsearch", "hello"]'

HTTP/1.1 200 OK
Content-Length: 74
Content-Type: application/json; charset=UTF-8

{
    "_id": "1", 
    "_index": "blog", 
    "_type": "article", 
    "_version": 2, 
    "created": false
}
```

#### 4、Delete a article
```python
article = Article.get(id=1)
article.delete()
```

=> Restful API
```
http DELETE http://127.0.0.1:9200/blog/article/1
HTTP/1.1 200 OK
Content-Length: 71
Content-Type: application/json; charset=UTF-8

{
    "_id": "1", 
    "_index": "blog", 
    "_type": "article", 
    "_version": 4, 
    "found": true
}

http HEAD  http://127.0.0.1:9200/blog/article/1
HTTP/1.1 404 Not Found
Content-Length: 0
Content-Type: text/plain; charset=UTF-8
```