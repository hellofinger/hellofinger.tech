---
title: Pandas基础
tags:
  - Python
  - Pandas
categories:
  - Python
author: Finger
date: 2018-08-24 22:19:00
---

原文地址：https://www.jiqizhixin.com/articles/2018-07-18-3

#### 介绍

Pandas是基于Numpy构建的库，在数据处理方面可以把它理解为numpy加强版。不同于numpy的是，pandas拥有种数据结构：**Series和DataFrame**：

![](/imgs/20180824/pandas-series.jpg)

用代码生成一个简单的Series对象

```
>>> from pandas import Series, DataFrame
>>> import pandas as pd
>>> data = Series([1,2,3,4],index = ['a','b','c','d'])
>>> data
a    1
b    2
c    3
d    4
dtype: int64
```

Series是一种类似一维数组的数据结构，由一组数据和与之相关的index组成，这个结构一看似乎与dict字典差不多，我们知道字典是一种无序的数据结构，而pandas中的Series的数据结构不一样，它相当于定长有序的字典，并且它的index和value之间是独立的，两者的索引还是有区别的，Series的index是可变的，而dict字典的key值是不可变的。

![](/imgs/20180824/pandas-dataframe.jpg)

用代码生成一个简单的DataFrame对象

```
>>> data = {'a':[1,2,3],'b':['we','you','they'],'c':['btc','eos','ae']}
>>> df = DataFrame(data)
>>> df
   a     b    c
0  1    we  btc
1  2   you  eos
2  3  they   ae
```


DataFrame这种数据结构我们可以把它看作是一张二维表，DataFrame长得跟我们平时使用的Excel表格差不多，DataFrame的横行称为columns，竖列和Series一样称为index，DataFrame每一列可以是不同类型的值集合，所以DataFrame你也可以把它视为不同数据类型同一index的Series集合。


#### Series

##### Series的两种生成方式
```
>>> data = Series([222,'btc',234,'eos'])
>>> data
0    222
1    btc
2    234
3    eos
dtype: object
```

虽然我们在生成的时候没有设置index值，但Series还是会自动帮我们生成index，这种方式生成的Series结构跟list列表差不多，可以把这种形式的Series理解为竖起来的list列表。

```
>>> data = Series([1,2,3,4],index = ['a','b','c','d'])
>>> data
a    1
b    2
c    3
d    4
dtype: int64
```

这种形式的Series可以理解为numpy的array外面披了一件index的马甲，所以array的相关操作，Series同样也是支持的。结构非常相似的dict字典同样也是可以转化为Series格式的：
```
>>> dic = {'a':1,'b':2,'c':'as'}
>>> dicSeries = Series(dic)
>>> dicSeries
a     1
b     2
c    as
dtype: object
```

##### 查看Series的相关信息
```
>>> data.index
Index(['a', 'b', 'c', 'd'], dtype='object')
>>> data.values
array([1, 2, 3, 4], dtype=int64)
# in方法默认判断的是index值
>>> 'a' in data
True
```

##### Series的NaN生成
```
>>> index1 = [ 'a','b','c','d']
>>> dic = {'b':1,'c':1,'d':1}
>>> data2 = Series(dic,index=index1)
>>> data2
a    NaN
b    1.0
c    1.0
d    1.0
dtype: float64
```

从这里我们可以看出Series的生成依据的是index值，index‘a’在字典dic的key中并不存在，Series自然也找不到’a’的对应value值，这种情况下Pandas就会自动生成NaN(not a number)来填补缺失值，这里还有个有趣的现象，原本dtype是int类型，生成NaN后就变成了float类型了，因为NaN的官方定义就是float类型。

##### NaN的相关查询
```
>>> data2.isnull()
a     True
b    False
c    False
d    False
dtype: bool
>>> data2.notnull()
a    False
b     True
c     True
d     True
dtype: bool

# 嵌套查询NaN
>>> data2[data2.isnull()==True]
a   NaN
dtype: float64

# 统计非NaN个数
>>> data2.count()
3

```

切记切记，查询NaN值切记不要使用np.nan==np.nan这种形式来作为判断条件，结果永远是False，==是用作值判断的，而NaN并没有值，如果你不想使用上方的判断方法，你可以使用is作为判断方法，is是对象引用判断，np.nan is np.nan，结果就是你要的True。

##### Series的name属性
```
>>> data.index.name = 'abc'
>>> data.name = 'test'
>>> data
abc
a    1
b    2
c    3
d    4
Name: test, dtype: int64
```

Series对象本身及其索引index都有一个name属性，name属性主要发挥作用是在DataFrame中，当我们把一个Series对象放进DataFrame中，新的列将根据我们的name属性对该列进行命名，如果我们没有给Series命名，DataFrame则会自动帮我们命名为0。

#### DataFrame

```
>>> data = {'name': ['BTC', 'ETH', 'EOS'], 'price':[50000, 4000, 150]}
>>> data = DataFrame(data)
>>> data
  name  price
0  BTC  50000
1  ETH   4000
2  EOS    150
```

DataFrame的生成与Series差不多，你可以自己指定index，也可不指定，DataFrame会自动帮你补上。

##### 查看DataFrame的相关信息
```
>>> data.index
RangeIndex(start=0, stop=3, step=1)
>>> data.values
array([['BTC', 50000],
       ['ETH', 4000],
       ['EOS', 150]], dtype=object)
>>> data.columns
Index(['name', 'price'], dtype='object')
```

##### DataFrame索引
```
>>> data.name
0    BTC
1    ETH
2    EOS
Name: name, dtype: object

>>> data['name']
0    BTC
1    ETH
2    EOS
Name: name, dtype: object

# loc['name']查询的是行标签
>>> data.iloc[1]
name      ETH
price    4000
Name: 1, dtype: object
```

其实行索引，除了iloc，loc还有个ix，ix既可以进行行标签索引，也可以进行行号索引，但这也大大增加了它的不确定性，有时会出现一些奇怪的问题，所以pandas在0.20.0版本的时候就把ix给弃用了。

##### DataFrame的常用操作

###### 简单地增加行、列
```
>>> data['type'] = 'token'
>>> data
  name  price   type
0  BTC  50000  token
1  ETH   4000  token
2  EOS    150  token

>>> data.loc['3'] = ['ae',200,'token']
>>> data
  name  price   type
0  BTC  50000  token
1  ETH   4000  token
2  EOS    150  token
3   ae    200  token
```

###### 删除行、列操作
```
>>> del data['type']
>>> data
  name  price
0  BTC  50000
1  ETH   4000
2  EOS    150
3   ae    200

>>> data.drop([2])
  name  price
0  BTC  50000
1  ETH   4000
3   ae    200

>>> data
  name  price
0  BTC  50000
1  ETH   4000
2  EOS    150
3   ae    200
```

这里需要注意的是，使用drop（）方法返回的是Copy而不是视图，要想真正在原数据里删除行，就要设置inplace=True
```
>>> data.drop([2],inplace=True)
>>> data
  name  price
0  BTC  50000
1  ETH   4000
3   ae    200
```

###### 设置某一列为index
```
>>> data.set_index(['name'],inplace=True)
>>> data
      price
name
BTC   50000
ETH    4000
ae      200

# 将index返回回dataframe中
>>> data.reset_index(inplace=True)
>>> data
  name  price
0  BTC  50000
1  ETH   4000
2   ae    200
```

###### 处理缺失值
```
data.dropna()    # 丢弃含有缺失值的行
data.fillna(0)    # 填充缺失值数据为0
```

还是需要注意：这些方法返回的是copy而不是视图，如果想在原数据上改变，别忘了inplace=True。

###### 数据合并
```
>>> data
  name  price
0  BTC  50000
1  ETH   4000
2   ae    200
3  eos    eos

>>> data1
  name other
0  BTC     a
1  ETH     b

# 以name为key进行左连接
>>> pd.merge(data,data1,on='name',how='left')
  name  price other
0  BTC  50000     a
1  ETH   4000     b
2   ae    200   NaN
3  eos    eos   NaN
```

平时进行数据合并操作，更多的会出一种情况，那就是出现重复值，DataFrame也为我们提供了简便的方法：

```
data.drop_duplicates(inplace=True)
```

###### 数据的简单保存与读取
```
>>> data.to_csv('test.csv')
>>> pd.read_csv('test.csv')
   Unnamed: 0 name  price
0           0  BTC  50000
1           1  ETH   4000
2           2   ae    200
3           3  eos    eos
```

为什么会出现这种情况呢，从头看到尾的同学可能就看出来了，增加第三行时，我用的是loc[‘3’]行标签来增加的，而read_csv方法是默认index是从0开始增长的，此时只需要我们设置下index参数就ok了

```
# 不保存行索引
>>> data.to_csv('test.csv',index=None)
>>> pd.read_csv('test.csv')
  name  price
0  BTC  50000
1  ETH   4000
2   ae    200
3  eos    eos
```

其他的还有header参数，这些参数都是我们在保存数据时需要注意的。
