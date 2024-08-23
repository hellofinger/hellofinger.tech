---
title: Using Elasticsearch Bulk With Python
tags:
  - ElasticSearch
  - SearchEngine
categories:
  - ElasticSearch
  - Storage
author: Finger
date: 2017-08-16 22:02:00
---


Python Elasticsearch客户端提供了批量索引的功能，详情请查看[项目主页](https://github.com/elastic/elasticsearch-py)

主要封装了两种方式进行`bulk`操作：

- streaming_bulk 	流式
- parallel_buck 	多线程

```python
def streaming_bulk(
	client, 
	actions, 
	chunk_size=500, 
	max_chunk_bytes=104857600, 
	raise_on_error=True, 
	expand_action_callback=<function expand_action>, 
	raise_on_exception=True, 
	max_retries=0, 
	initial_backoff=2, 
	max_backoff=600, 
	yield_ok=True, 
	**kwargs): pass


def parallel_bulk(
	client, 
	actions, 
	thread_count=4, 
	chunk_size=500, 
	max_chunk_bytes=104857600, 
	queue_size=4, 
	expand_action_callback=<function expand_action>, 
	**kwargs): pass
```

`streaming`比`parallel`多了重试的机制。两个方法默认都是每次处理500索引，两个方法都返回迭代器对象，`bulk`方法默认使用的是`streaming_bulk`方式。如果要使用`parallel_buck`需要修改源代码或者自己扩展。


bulk方法定义：
```python
def bulk(client, actions, stats_only=False, **kwargs):
    success, failed = 0, 0

    # list of errors to be collected is not stats_only
    errors = []

    for ok, item in streaming_bulk(client, actions, **kwargs):
        # go through request-reponse pairs and detect failures
        if not ok:
            if not stats_only:
                errors.append(item)
            failed += 1
        else:
            success += 1

    return success, failed if stats_only else errors
```
- client 	为ElasticSearch实例对象
- actions 	索引内容
- stats_only 为True则只返回成功和失败的数量，False则会返回具体的失败内容列表

actions 内容为：
```
{
    '_op_type': 'delete',
    '_index': 'index-name',
    '_type': 'type-name',
    '_id': 1,
}
{
    '_op_type': 'update',
    '_index': 'index-name',
    '_type': 'type-name',
    '_id': 1,
}
```
- `_op_type` 可以是`index`, `create`, `delete`, `update`操作，默认使用`index`

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

from elasticsearch import Elasticsearch
from elasticsearch import helpers

def test_streaming_bulk():
    actions = []
    for i in xrange(0, 100000):
        actions.append({'_index': 'test_index', '_type': 'test', 'x': i})

    helpers.bulk(es, actions, False)


def test_parallel_bulk():
    actions = []
    for i in xrange(0, 100000):
        actions.append({'_index': 'test_index', '_type': 'test', 'x': i})

    parallel_bulk(es, actions)


def parallel_bulk(client, actions, stats_only=False, **kwargs):

    success, failed = 0, 0

    # list of errors to be collected is not stats_only
    errors = []

    for ok, item in helpers.parallel_bulk(client, actions, **kwargs):
        # print ok, item
        # go through request-reponse pairs and detect failures
        if not ok:
            if not stats_only:
                errors.append(item)
            failed += 1
        else:
            success += 1

    return success, failed if stats_only else errors

```

测试代码： https://github.com/zhoujun/mydemos/tree/master/elasticsearch-demo
