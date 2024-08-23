---
title: Python中的10个隐藏特性
author: Finger
tags:
  - Python
categories:
  - Python
date: 2018-05-24 11:10:00
---


#### 1、函数参数解包(unpack)

```
>>> def foo(x, y): print(x, y)
... 
>>> alist = [1,2]
>>> adict = {'x': 1, 'y': 2}
>>> foo(*alist)
1 2
>>> foo(**adict)
1 2
>>> 
```

#### 2、python3中的元组unpack

```
>>> a, b, *rest = range(10)
>>> a
0
>>> b
1
>>> rest
[2, 3, 4, 5, 6, 7, 8, 9]
>>>
```

也可以取最后一个：

```
>>> first, second, *rest, last = range(10)
>>> first
0
>>> second
1
>>> last
9
>>> rest
[2, 3, 4, 5, 6, 7, 8]
```

#### 3、链式比较操作符

```
>>> 1 < x < 2
False
>>> 4 > x >= 3
True
>>>
```

#### 4、列表切片操作

``` 
# 按步长2取列表数据
>>> a = [1,2,3,4,5]
>>> a[::2] 
[1, 3, 5]

# 用步长-1来反转列表
>>> a[::-1]
[5, 4, 3, 2, 1]

# 用切片来删除列表的某一段
>>> a = [1,2,3,4,5]
>>> a[1:3] = []
>>> a
[1, 4, 5]

# 也可以用 del a[1:3]
>>> a = [1,2,3,4,5]
>>> del a[1:3]
>>> a
[1, 4, 5]

```

#### 5、嵌套的列表推导式

```
>>> [(i, j) for i in range(3) for j in range(i)]
[(1, 0), (2, 0), (2, 1)]
```

#### 6、字典里的无限递归

```
>>> a, b = {}, {}
>>> a['b'] = b
>>> b['a'] = a
>>> a
{'b': {'a': {...}}}
```

当然列表也可以

```
>>> a, b = [], []
>>> a.append(b)
>>> b.append(a)
>>> a
[[[...]]]
```

#### 7、下划线"_"

```
# _ 在Python解析器上返回上一次的值

>>> 1 + 1
2
>>> _
2
```

另外 Python中的"_"也经常用在未使用的变量命名

```
>>> pos = (1, 2)
>>> x, _ = pos
```

#### 8、注意函数的默认参数

```
>>> def foo(x=[]):
...     x.append(1)
...     print x
...
>>> foo()
[1]
>>> foo()
[1, 1]
```

更安全的做法：

```
>>> def foo(x=None):
...     if x is None:
...         x = []
...     x.append(1)
...     print x
...
>>> foo()
[1]
>>> foo()
[1]
>>>
```

#### 9、另一种字符串连接

```
>>> name = "Hello" "World"
>>> name
'HelloWorld'
```

连接多行：

```
>>> name = "Hello" \
...     "World"
>>> name
'HelloWorld'
```

#### 10、不能访问到的属性

```
>>> class Foo(object): pass
...
>>> obj = Foo()
>>> setattr(o, 'hello world', 1)
>>> o.hello world
  File "<stdin>", line 1
    o.hello world
                ^
SyntaxError: invalid syntax
```

不过，能用 `setattr` 设置属性，就可以用 `getattr` 取出

```
>>> getattr(o, 'hello world')
1
```







