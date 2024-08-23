---
title: Elasticsearch Documents APIs
tags:
  - ElasticSearch
  - SearchEngine
categories:
  - ElasticSearch
  - Storage
author: Finger
date: 2017-08-15 23:24:00
---


### 一、Elasticsearch 索引别名
索引别名就像一个快捷方式或软链接，可以指向一个或多个索引，也可以给任何需要索引名的API使用。别名带给我们极大的灵活性，允许我们做到：
- 在一个运行的集群上无缝的从一个索引切换到另一个
- 给多个索引分类
- 给索引的一个子集创建视图


```
# 创建别名
PUT /my_index_v1 

# 将别名my_index指向my_index_v1
PUT /my_index_v1/_alias/my_index 

# 检查别名指向哪个索引
GET /*/_alias/my_index

# 或者哪个别名指向这个索引
GET /my_index_v1/_alias/*
```

如果我们决定要修改索引中一个字段的映射。但是我们不能修改现存的映射，因此我们需要重新索引数据。这个时候别名就可以帮助我们完成。

首先，我们需要创建新的索引 `my_index_v2`
```
PUT /my_index_v2
```

然后我们将数据从 `my_index_v1` 迁移到 `my_index_v2`
```
POST /_aliases
{
    "actions": [
        { "remove": { "index": "my_index_v1", "alias": "my_index" }},
        { "add":    { "index": "my_index_v2", "alias": "my_index" }}
    ]
}
```
最后，我们的应用就从旧的索引迁移到了新的，而没有停机时间。

**提示：**
> 即使你认为现在的索引设计已经是完美的了，当你的应用在生产环境使用时，还是有可能在今后有一些改变的。
索引请做好准备：在应用中使用别名而不是索引。然后你就可以在任何时候重新索引。别名的开销很小，应当广泛使用。

### 二、Elasticsearch 文档APIs 

#### 1、创建文档
```
# 创建index为twitter, type为tweet，id为1的文档
PUT twitter/tweet/1
{
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}

# 结果为：
{
    "_shards" : {
        "total" : 2,
        "failed" : 0,
        "successful" : 2
    },
    "_index" : "twitter",
    "_type" : "tweet",
    "_id" : "1",
    "_version" : 1,
    "created" : true,
    "result" : created
}
```

#### 2、获取文档
```
GET twitter/tweet/1

{
    "_index" : "twitter",
    "_type" : "tweet",
    "_id" : "0",
    "_version" : 1,
    "found": true,
    "_source" : {
        "user" : "kimchy",
        "date" : "2009-11-15T14:12:12",
        "likes": 0,
        "message" : "trying out Elasticsearch"
    }
}
```

#### 3、删除文档
```
DELETE /twitter/tweet/1

{
    "_shards" : {
        "total" : 2,
        "failed" : 0,
        "successful" : 2
    },
    "found" : true,
    "_index" : "twitter",
    "_type" : "tweet",
    "_id" : "1",
    "_version" : 2,
    "result": "deleted"
}
```

#### 4、删除Type映射

报错：
No handler found for uri [/edemo/test] and method [DELETE]

因为ElasticSearch已经不支持删除一个type了。所以如果想要删除type有两种选择：
- 重新设置index
- 删除type下的所有内容

1、如果是重新设置index的话，官方建议尽可能不要直接删除type。应该是删除index然后重新创建type。
具体请查看[官方文档](https://www.elastic.co/guide/en/elasticsearch/reference/5.4/indices-delete-mapping.html)

2、如果是想要删除type下的所有数据的话，可以使用delete by query的方法:
```
POST index/type/_delete_by_query?conflicts=proceed
{
  "query": {
    "match_all": {}
  }
}
```
具体请查看[官方文档](https://www.elastic.co/guide/en/elasticsearch/reference/5.4/docs-delete-by-query.html)

#### 5、查看映射(Mapping)
```
# 查看index中的所有mapping
GET /index/_mapping

# 查看index中某一个type的mapping
GET /index/_mapping/type

```

#### 6、批量操作(Bulk)
```
POST _bulk

{ "index" : { "_index" : "test", "_type" : "type1", "_id" : "1" } }
{ "field1" : "value1" }
{ "delete" : { "_index" : "test", "_type" : "type1", "_id" : "2" } }
{ "create" : { "_index" : "test", "_type" : "type1", "_id" : "3" } }
{ "field1" : "value3" }
{ "update" : {"_id" : "1", "_type" : "type1", "_index" : "test"} }
{ "doc" : {"field2" : "value2"} }

# 结果为：
{
   "took": 30,
   "errors": false,
   "items": [
      {
         "index": {
            "_index": "test",
            "_type": "type1",
            "_id": "1",
            "_version": 1,
            "result": "created",
            "_shards": {
               "total": 2,
               "successful": 1,
               "failed": 0
            },
            "created": true,
            "status": 201
         }
      },
      {
         "delete": {
            "found": false,
            "_index": "test",
            "_type": "type1",
            "_id": "2",
            "_version": 1,
            "result": "not_found",
            "_shards": {
               "total": 2,
               "successful": 1,
               "failed": 0
            },
            "status": 404
         }
      },
      {
         "create": {
            "_index": "test",
            "_type": "type1",
            "_id": "3",
            "_version": 1,
            "result": "created",
            "_shards": {
               "total": 2,
               "successful": 1,
               "failed": 0
            },
            "created": true,
            "status": 201
         }
      },
      {
         "update": {
            "_index": "test",
            "_type": "type1",
            "_id": "1",
            "_version": 2,
            "result": "updated",
            "_shards": {
                "total": 2,
                "successful": 1,
                "failed": 0
            },
            "status": 200
         }
      }
   ]
}
```
#### 7、其他
- [Update API](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-update.html)
- [Update By Query API](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-update-by-query.html)
- [Multi Get API](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-multi-get.html)
- [Reindex API](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-reindex.html)


### 参考文档
https://www.elastic.co/guide/en/elasticsearch/reference/current/docs.html

http://blog.csdn.net/leafage_m/article/details/74011357

