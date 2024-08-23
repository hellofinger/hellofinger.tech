---
title: 用Python实现Elasticsearch function_score查询(一)
tags:
  - ElasticSearch
  - Python
categories:
  - ElasticSearch
  - Storage
author: Finger
date: 2017-08-19 12:38:00
---

### 什么是function_score

`function_score`查询是处理分值计算过程的终极工具。它让你能够对所有匹配了主查询的每份文档调用一个函数来调整甚至是完全替换原来的`_score`。

实际上，你可以通过设置过滤器来将查询得到的结果分成若干个子集，然后对每个子集使用不同的函数。这样你就能够同时得益于：高效的分值计算以及可以缓存的过滤器。

它拥有几种预先定义好了的函数：

- `weight` 对每份文档适用一个简单的提升，且该提升不会被归约：当`weight`为2时，结果为2 * `_score`。
- `field_value_factor` 使用文档中某个字段的值来改变`_score`，比如将受欢迎程度或者投票数量考虑在内。
- `random_score` 使用一致性随机分值计算来对每个用户采用不同的结果排序方式，对相同用户仍然使用相同的排序方式。
- 衰减函数(Decay Function)-(`linear`，`exp`，`gauss`) 将像`publish_date`，`geo_location`或者`price`这类浮动值考虑到`_score`中，偏好最近发布的文档，邻近于某个地理位置(译注：其中的某个字段)的文档或者价格(译注：其中的某个字段)靠近某一点的文档。
- `script_score` 使用自定义的脚本来完全控制分值计算逻辑。如果你需要以上预定义函数之外的功能，可以根据需要通过脚本进行实现。

没有`function_score`查询的话，我们也许就不能将全文搜索得到分值和近因进行结合了。我们将不得不根据`_score`或者`date`进行排序；无论采用哪一种都会抹去另一种的影响。`function_score`查询让我们能够将两者融合在一起：仍然通过全文相关度排序，但是给新近发布的文档，或者流行的文档，或者符合用户价格期望的文档额外的权重。你可以想象，一个拥有所有这些功能的查询看起来会相当复杂。我们从一个简单的例子开始，循序渐进地对它进行介绍。

### 如何使用function_score

假设我们有一个博客网站让用户投票选择他们喜欢的文章。我们希望让人气高的文章出现在结果列表的头部，但是主要的排序依据仍然是全文搜索分值。我们可以通过保存每篇文章的投票数量来实现。

```python
# 1、新建一个索引映射：
class Post(DocType):
    title = String()
    content = Text()
    votes = Integer()
    created_at = Date()

    class Meta:
        index = 'blogposts'
        type = 'post'

Post.init()

# 2、添加测试数据
def add_post(id_, title, content):
    post = Post(meta={'id': id_})
    post.title = title
    post.content = content
    post.votes = random.randint(0, 100)
    post.save()

# 3、初始化数据
def init_data():
    add_post(1, 'Python is good', 'Python is good')
    add_post(2, 'Python is beautiful', 'Python is beautiful')
    add_post(3, 'Python is nice', 'Python is nice')
    add_post(4, 'Today is good day', 'Today is good day')
    add_post(5, 'Today is nice day', 'Today is nice day')
    add_post(6, 'The key aspect in promoting', 'The key aspect in promoting')
    add_post(7, 'Just a month after we started', 'Just a month after we started working on reddit')
    add_post(8, "If I can't hear your heartbeat", "If I can't hear your heartbeat, you're too far away.")
    add_post(9, 'No Country for Old Men', 'No Country for Old Men')
    add_post(10, 'Django is web framework', 'Django is python lib')

```

#### field_value_factor

使用`field_value_factor`函数的`function_score`查询将投票数和全文相关度分值结合起来：
```python
q = query.Q(
    'function_score',
    query=query.Q("multi_match", query='python', fields=['title', 'content']),
    functions=[
        query.SF('field_value_factor', field='votes')
    ]
)
s = Post.search()
s = s.query(q)
response = s.execute()
for h in response:
    print(h.title)

"""
Out:
Python is nice
Python is good
Python is beautiful
Django is web framework
"""
```
![](/imgs/20170819/20170819125817.png)

`function_score`查询会包含主查询(Main Query)和希望使用的函数。先会执行主查询，然后再为匹配的文档调用相应的函数。每份文档中都必须有一个`votes`字段用来保证`function_score`能够起作用。

在前面的例子中，每份文档的最终`_score`会通过下面的方式改变：
> new_score = old_score * number_of_votes

它得到的结果并不好。全文搜索的`_score`通常会在0到10之间。而从下图我们可以发现，拥有10票以上的文章的分值大大超过了这个范围，而没有被投票的文章的分值会被重置为0。
![](/imgs/20170819/20170819125818.png)
![](/imgs/20170819/20170819133637.png)

#### modifier

为了让`votes`值对最终分值的影响更缓和，我们可以使用modifier。换言之，我们需要让头几票的效果更明显，其后的票的影响逐渐减小。0票和1票的区别应该比10票和11票的区别要大的多。

一个用于此场景的典型modifier是log1p，它将公式改成这样：
> new_score = old_score * log(1 + number_of_votes)

`log`函数将`votes`字段的效果减缓了，其效果类似下面的曲线：
![](/imgs/20170819/20170819125819.png)
使用了`modifier`参数的请求如下：
```python
q = query.Q(
    'function_score',
    query=query.Q("multi_match", query='python', fields=['title', 'content']),
    functions=[
        query.SF('field_value_factor', field='votes', modifier='log1p')
    ]
)
```
![](/imgs/20170819/20170819131048.png)
可用的`modifiers`有：none(默认值)，log，log1p，log2p，ln，ln1p，ln2p，square，sqrt以及reciprocal。

Modifier|Meaning 
----|-----
none    | Do not apply any multiplier to the field value
log     | Take the logarithm of the field value 
log1p   | Add 1 to the field value and take the logarithm
log2p   | Add 2 to the field value and take the logarithm
ln      | Take the natural logarithm of the field value
ln1p    | Add 1 to the field value and take the natural logarithm
ln2p    | Add 2 to the field value and take the natural logarithm
square  | Square the field value (multiply it by itself)
sqrt	| Take the square root of the field value
reciprocal| Reciprocate the field value, same as 1/x where x is the field’s value

具体详情请查阅[官网介绍](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-function-score-query.html#function-field-value-factor)

#### factor
可以通过将`votes`字段的值乘以某个数值来增加该字段的影响力，这个数值被称为`factor`：
```python
q = query.Q(
    'function_score',
    query=query.Q("multi_match", query='python', fields=['title', 'content']),
    functions=[
        query.SF('field_value_factor', field='votes', modifier='log1p', factor=2)
    ]
)
```
添加了`factor`将公式修改成这样：
>new_score = old_score * log(1 + factor * number_of_votes)
当`factor`大于1时，会增加其影响力，而小于1的`factor`则相应减小了其影响力，如下图所示：

![](/imgs/20170819/20170819131049.png)
![](/imgs/20170819/20170819132115.png)

#### boost_mode
将全文搜索的相关度分值乘以`field_value_factor`函数的结果，对最终分值的影响可能太大了。通过`boost_mode`参数，我们可以控制函数的结果应该如何与`_score`结合在一起，该参数接受下面的值：

模型|描述 
----|-----
multiply    | _score 与函数值的积（默认）
sum         | _score 与函数值的和  
avg         | _score 平均值  
first       |应用filter匹配上的第一个函数值
min         |_score 与函数值间的较小值
max         | _score 与函数值间的较大值
replace     | 将_score替换成函数结果

如果我们是通过将函数结果累加来得到`_score`，其影响会小的多，特别是当我们使用了一个较低的`factor`时：
```python
q = query.Q(
    'function_score',
    query=query.Q("multi_match", query='python', fields=['title', 'content']),
    functions=[
        query.SF('field_value_factor', field='votes', modifier='log1p', factor=0.1)
    ],
    score_mode="sum"
)
```
上述请求的公式如下所示：
> new_score = old_score + log(1 + 0.1 * number_of_votes)

![](/imgs/20170819/20170819131050.png)
![](/imgs/20170819/20170819133115.png)

#### max_boost
最后，我们能够通过制定`max_boost`参数来限制函数的最大影响：
```python
q = query.Q(
    'function_score',
    query=query.Q("multi_match", query='python', fields=['title', 'content']),
    functions=[
        query.SF('field_value_factor', field='votes', modifier='log1p', factor=0.1)
    ],
    score_mode="sum",
    max_boost=1.5
)
```

无论`field_value_factor`函数的结果是多少，它绝不会大于1.5。

NOTE
> max_boost只是对函数的结果有所限制，并不是最终的_score。

![](/imgs/20170819/20170819134205.png)

### 参考文档

https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-function-score-query.html

http://itindex.net/detail/57059-elasticsearch-%E6%8E%A7%E5%88%B6-%E7%9B%B8%E5%85%B3