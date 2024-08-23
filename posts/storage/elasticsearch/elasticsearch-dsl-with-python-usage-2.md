---
title: Python Elasticsearch DSL 使用笔记(二)
tags:
  - Python
  - ElasticSearch
  - SearchEngine
categories:
  - ElasticSearch
  - Storage
author: Finger
date: 2017-08-13 07:56:00
---


### 使用ElasticSearch DSL进行搜索

Search主要包括：
- 查询(queries)
- 过滤器(filters)
- 聚合(aggreations)
- 排序(sort)
- 分页(pagination)
- 额外的参数(additional parameters)
- 相关性(associated)

创建一个查询对象
```python
from elasticsearch import Elasticsearch
from elasticsearch_dsl import Search

client = Elasticsearch()

s = Search(using=client)
```

初始化测试数据
```python

def add_article(id_, title, body, tags):
    article = Article(meta={'id': id_}, title=title, tags=tags)
    article.body = body
    article.published_from = datetime.now()
    article.save()


def init_test_data():
    add_article(2, 'Python is good!', 'Python is good!', ['python'])
    add_article(3, 'Elasticsearch', 'Distributed, open source search and analytics engine', ['elasticsearch'])
    add_article(4, 'Python very quickly', 'Python very quickly', ['python'])
    add_article(5, 'Django', 'Python Web framework', ['python', 'django'])
```

第一个查询语句
```python
# 创建一个查询语句
s = Search().using(client).query("match", title="python")

# 查看查询语句对应的字典结构
print(s.to_dict())
# {'query': {'match': {'title': 'python'}}}

# 发送查询请求到Elasticsearch
response = s.execute()

# 打印查询结果
for hit in s:
    print(hit.title)

# Out:
Python is good!
Python very quickly

# 删除查询
s.delete()
```

### 1、Queries

```python
# 创建一个多字段查询
multi_match = MultiMatch(query='python', fields=['title', 'body'])
s = Search().query(multi_match)
print(s.to_dict())
# {'query': {'multi_match': {'fields': ['title', 'body'], 'query': 'python'}}}

# 使用Q语句
q = Q("multi_match", query='python', fields=['title', 'body'])
# 或者
q = Q({"multi_match": {"query": "python", "fields": ["title", "body"]}})

s = Search().query(q)
print(s.to_dict())

# If you already have a query object, or a dict 
# representing one, you can just override the query used 
# in the Search object:
s.query = Q('bool', must=[Q('match', title='python'), Q('match', body='best')])
print(s.to_dict())


# 查询组合
q = Q("match", title='python') | Q("match", title='django')
s = Search().query(q)
print(s.to_dict())
# {"bool": {"should": [...]}}

q = Q("match", title='python') & Q("match", title='django')
s = Search().query(q)
print(s.to_dict())
# {"bool": {"must": [...]}}

q = ~Q("match", title="python")
s = Search().query(q)
print(s.to_dict())
# {"bool": {"must_not": [...]}}
```

### 2、Filters

```python
s = Search()
s = s.filter('terms', tags=['search', 'python'])
print(s.to_dict())
# {'query': {'bool': {'filter': [{'terms': {'tags': ['search', 'python']}}]}}}

s = s.query('bool', filter=[Q('terms', tags=['search', 'python'])])
print(s.to_dict())
# {'query': {'bool': {'filter': [{'terms': {'tags': ['search', 'python']}}]}}}

s = s.exclude('terms', tags=['search', 'python'])
# 或者
s = s.query('bool', filter=[~Q('terms', tags=['search', 'python'])])
print(s.to_dict())
# {'query': {'bool': {'filter': [{'bool': {'must_not': [{'terms': {'tags': ['search', 'python']}}]}}]}}}
```

### 3、Aggregations

```python
s = Search()
a = A('terms', filed='title')
s.aggs.bucket('title_terms', a)
print(s.to_dict())
# {
# 'query': {
#   'match_all': {}
#  },
#  'aggs': {
#       'title_terms': {
#            'terms': {'filed': 'title'}
#        }
#    }
# }

# 或者
s = Search()
s.aggs.bucket('articles_per_day', 'date_histogram', field='publish_date', interval='day') \
    .metric('clicks_per_day', 'sum', field='clicks') \
    .pipeline('moving_click_average', 'moving_avg', buckets_path='clicks_per_day') \
    .bucket('tags_per_day', 'terms', field='tags')

s.to_dict()
# {
#   "aggs": {
#     "articles_per_day": {
#       "date_histogram": { "interval": "day", "field": "publish_date" },
#       "aggs": {
#         "clicks_per_day": { "sum": { "field": "clicks" } },
#         "moving_click_average": { "moving_avg": { "buckets_path": "clicks_per_day" } },
#         "tags_per_day": { "terms": { "field": "tags" } }
#       }
#     }
#   }
# }
```

### 4、Sorting
```python
s = Search().sort(
    'category',
    '-title',
    {"lines" : {"order" : "asc", "mode" : "avg"}}
)
```

### 5、Pagination
```python 
s = s[10:20]
# {"from": 10, "size": 10}
```

### 6、Extra Properties and parameters
```python 
s = Search()
# 设置扩展属性使用`.extra()`方法
s = s.extra(explain=True)

# 设置参数使用`.params()`
s = s.params(search_type="count")

# 如要要限制返回字段，可以使用`source()`方法
# only return the selected fields
s = s.source(['title', 'body'])
# don't return any fields, just the metadata
s = s.source(False)
# explicitly include/exclude fields
s = s.source(include=["title"], exclude=["user.*"])
# reset the field selection
s = s.source(None)

# 使用dict序列化一个查询
s = Search.from_dict({"query": {"match": {"title": "python"}}})

# 修改已经存在的查询
s.update_from_dict({"query": {"match": {"title": "python"}}, "size": 42})
```

**测试代码**： https://github.com/zhoujun/mydemos/tree/master/elasticsearch-demo