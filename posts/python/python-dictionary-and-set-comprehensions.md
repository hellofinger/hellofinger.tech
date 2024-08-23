---
title: Python Dictionary and Set comprehensions
author: Finger
tags:
  - Python
categories:
  - Python
date: 2018-05-21 10:40:00
---


大多数Python程序员都知道并使用列表推导。对于不熟悉这个概念的人来说，列表推导其实是一种更简短、更简洁的创建列表的方式。

```
>>> some_list = [1, 2, 3, 4, 5]
>>> another_list = [ x + 1 for x in some_list ]
>>> another_list
[2, 3, 4, 5, 6]

```

而从Python 3.1开始(也反向地移植到了Python 2.7), 我们可以用同样的方式创建集合(Set)和字典(Dict):

使用推导的方式，创建一个集合

```
>>> # Set Comprehensions
>>> some_list = [1, 2, 3, 4, 5, 2, 5, 1, 4, 8]
>>> even_set = { x for x in some_list if x % 2 == 0 }
>>> even_set
set([8, 2, 4])
```

使用推导的方式，创建一个Dict

```
>>> # Dict Comprehensions
>>> d = { x: x % 2 == 0 for x in range(1, 11) }
>>> d
{1: False, 2: True, 3: False, 4: True, 5: False, 6: True, 7: False, 8: True, 9: False, 10: True}
```

另一个创建集合的方式:

```
>>> my_set = {1, 2, 1, 2, 3, 4}
>>> my_set
set([1, 2, 3, 4])
```

没有使用到内建的 `set`方法, 直接用大括号{}创建。
