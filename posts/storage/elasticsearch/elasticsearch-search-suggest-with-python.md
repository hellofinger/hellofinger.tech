---
title: 用Python实现Elasticsearch提供的搜索建议
tags:
  - ElasticSearch
  - SearchEngine
categories:
  - ElasticSearch
  - Storage
author: Finger
date: 2017-08-16 23:01:00
---


### Suggest搜索

`Elasticsearch`支持多种`suggest`类型，也支持模糊搜索。

`Elasticsearch`支持如下4种搜索类型：

- Term。基于编辑距离的搜索，也就是对比两个字串之间，由一个转成另一个所需的最少编辑操作次数，编辑距离越少说明越相近。搜索时需要指定字段，是很基础的一种搜索。
- Phrase。Term的优化，能够基于共现和频率来做出关于选择哪些token的更好的决定。
- Completion。提供自动完成/按需搜索的功能，这是一种导航功能，可在用户输入时引导用户查看相关结果，从而提高搜索精度。和前2种用法不同，需要在mapping时指定suggester字段，使用允许快速查找的数据结构，所以在搜索速度上得到了很大的优化。
- Context。Completion搜索的是索引中的全部文档，但是有时候希望对这个结果进行一些and/or的过滤，就需要使用Context类型了。

下面使用[elasticsearch_dsl](https://github.com/elastic/elasticsearch-dsl-py)自带的`Suggestions`功能
```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

from elasticsearch_dsl import DocType, Text, Keyword, Completion
from elasticsearch_dsl.connections import connections

connections.create_connection(hosts=['localhost'])


class BlogLive(DocType):
    subject = Text()
    description = Text()
    topics = Keyword()
    live_suggest = Completion()

    class Meta:
        index = 'blog'
        type = 'live'

    def save(self, ** kwargs):
        return super(BlogLive, self).save(** kwargs)


# create the mappings in elasticsearch
BlogLive.init()


def add_live(id_, subject, description):
    live = BlogLive(meta={'id': id_})
    live.subject = subject
    live.description = description
    live.live_suggest = subject
    live.save()


def init_data():
    add_live(1, 'python', 'python is good')
    add_live(2, 'python django', 'django is web framework')
    add_live(3, 'python flask', 'flask is web framework')
    add_live(4, 'elasticsearch', 'elasticsearch is searchengine')
    add_live(5, 'elasticsearch-dsl python', 'python elasticsearch dsl is client API')


def suggest(key):
    s = BlogLive.search()
    s = s.suggest('live_suggestion', key,
                  completion={'field': 'live_suggest', 'fuzzy': {'fuzziness': 2, 'prefix_length': 2}, 'size': 10})
    print(s.to_dict())
    """
    {'suggest': 
        {'live_suggestion': 
             {
                'completion': {'field': 'live_suggest', 'size': 10, 'fuzzy': {'fuzziness': 2, 'prefix_length': 2}}, 
                'text': 'python'
            }
        }
    }
    """
    suggestions = s.execute_suggest()
    # print(suggestions.live_suggestion)
    for match in suggestions.live_suggestion[0].options:
        source = match._source
        print(source['subject'],  source['description'], match._score)


if __name__ == '__main__':
    init_data()
    suggest('python')
```