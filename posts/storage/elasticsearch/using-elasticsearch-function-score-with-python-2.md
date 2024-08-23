---
title: 用Python实现Elasticsearch function_score查询(二)
tags:
  - Python
  - ElasticSearch
categories:
  - ElasticSearch
  - Storage
author: Finger
date: 2017-08-26 10:23:00
---


#### random_score

`random_score`函数，它的输出是一个介于0到1之间的数字，当给它提供相同的`seed`值时，它能够产生一致性随机的结果。`random_score` 子句不包含任何的`filter`，因此它适用于所有文档。当然，如果你索引了能匹配查询的新文档，无论你是否使用了一致性随机，结果的顺序都会有所改变。

```python
q = query.Q(
    'function_score',
     functions=[
        query.SF('random_score', seed=10)
    ]
)
```

```
GET /_search
{
    "query": {
        "function_score": {
            "random_score": {
                "seed": 10
            }
        }
    }
}
```
#### decay functions
之前提过的`weight`，`field_value_factor`，`random_score`等函数都是一种线性的影响关系，给出的分数随着距离变大而变低。有时候我们会有范围性质，分数变化缓慢的需求场景。此时上述函数就显得不那么适用。
`Elasticsearch`提供了三种`decay functions`，让我们可以基于一个单值的数值型字段（比如日期、地点或者价格类似标准的数值型字段）来计算分数。

这三个衰减函数分别为：
- 线性衰减(linear)
- 指数衰减(exp)
- 高斯衰减(gauss)

它们可以操作数值，时间以及经纬度地理坐标点这样的字段，这三个函数都有以下参数：
- origin: 中心点或字段可能的最佳值，落在原点`origin`上的文档评分 _score 为满分1.0 。
- scale:  衰减率，即一个文档从原点origin下落时，评分`_score`改变的速度(例如每 £10 欧元或每 100 米)。
- decay:  从原点`origin`衰减到`scale`所得的评分`_score`，默认值为 0.5。
- offset: 以原点`origin`为中心点，为其设置一个非零的偏移量offset 覆盖一个范围，而不只是单个原点。在范围 -offset <= origin <= +offset 内的所有评分`_score`都是 1.0。

`origin`和`scale`参数是必须的。`origin`是中心点，计算是从这点开始。`scale`是衰变率。默认情况下，`offset`参数设置为0；如果定义了该参数，`decay`函数将只计算文档值比此参数的值大的文档得分。`decay`参数告诉Elasticsearch应该降低多少分数，默认设置为0.5。

下面我们举一个例子：假设用户希望租一个离伦敦市中心({'lat':51.50, 'lon':0.12})且每晚不超过 £100 英镑的度假屋而且与距离相比，用户对价格更为敏感，这样查询可以写成：
```python
q = query.Q(
    'function_score',
    query=query.Q('match', properties="balcony"),
    functions=[
        query.SF('gauss', price={'origin': '0', 'scale': '20'}),
        query.SF('gauss', location={'origin': '51.50, 0.12', 'scale': '2km'})
    ],
    score_mode="multiply"
)
```

```
{  
    "query": {  
        "function_score": {  
          "functions": [  
            {  
              "gauss": {  
                "price": {  
                  "origin": "0",  
                  "scale": "20"  
                }  
              }  
            },  
            {  
              "gauss": {  
                "location": {  
                  "origin": "11, 12",  
                  "scale": "2km"  
                }  
              }  
            }  
          ],  
          "query": {  
            "match": {  
              "properties": "balcony"  
            }  
          },  
          "score_mode": "multiply"  
        }  
    }  
}  
```

关于decay functions的衰减曲线性：如下图：
![](/imgs/20170826/decay_2d.png)

图中所有曲线的原点`origin`（即中心点）的值都是40，`offset`是5，也就是在范围 40 - 5 <= value <= 40 + 5 内的所有值都会被当作原点`origin`处理所有这些点的评分都是满分1.0 。
在此范围之外，评分开始衰减，衰减率由`scale`值（此例中的值为 5 ）和 衰减值`decay`此例中为默认值 0.5 ）共同决定。结果是所有三个曲线在 origin +/- (offset + scale) 处的评分都是 0.5，即点30和50处。

`linear` 、 `exp` 和 `gauss`函数三者之间的区别在于范围（ origin +/- (offset + scale) ）之外的曲线形状：

- linear 线性函数是条直线，一旦直线与横轴 0 相交，所有其他值的评分都是 0.0 。
- exp 指数函数是先剧烈衰减然后变缓。
- gauss 高斯函数是钟形的——它的衰减速率是先缓慢，然后变快，最后又放缓。

**选择曲线的依据完全由期望评分`_score`的衰减速率来决定，即距原点`origin`的值。**

Demo地址：https://github.com/zhoujun/mydemos/tree/master/elasticsearch-demo