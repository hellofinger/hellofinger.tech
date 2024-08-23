---
title: Python类的__dict__无法更新
author: Finger
tags:
  - Python
categories:
  - Python
date: 2018-05-17 20:40:00
---

### Python类的__dict__无法更新？

测试环境: Python2.7

```python
>>> class O(object): pass
...
>>> O.__dict__['name'] = 'Finger'
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: 'dictproxy' object does not support item assignment
```
答：是的, class的`__dict__`是只读的

```python
>>> O.__dict__
dict_proxy({'__dict__': <attribute '__dict__' of 'O' objects>, '__module__': '__main__', '__weakref__': <attribute '__weakref__' of 'O' objects>, '__doc__': None})

>>> O.__dict__.items()
[('__dict__', <attribute '__dict__' of 'O' objects>), ('__module__', '__main__'), ('__weakref__', <attribute '__weakref__' of 'O' objects>), ('__doc__', None)]

```

可以看到`O.__dict__` 是一个`dictproxy`对象，而不是一个`dict`。

```python
>>> dir(O.__dict__)
['__class__', '__cmp__', '__contains__', '__delattr__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__getitem__', '__gt__', '__hash__', '__init__', '__iter__', '__le__', '__len__', '__lt__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', 'copy', 'get', 'has_key', 'items', 'iteritems', 'iterkeys', 'itervalues', 'keys', 'values']
```
我们`dir(O.__dict__)`，发现它没有`__setitem__`方法

那么我们怎么改类设置属性呢？可以用`setattr`

```python
>>> setattr(O, 'name', 'Finger')
>>> O.name
'Finger'
>>>
```

可能在测试的过程中你会发现，如果你采用Python经典继承(Old-Style Classes)看到的结果却是这样的：

```python
>>> class O: pass
...
>>> O.__dict__['name'] = 'Finger'
>>> O.name
'Finger'
```
> 关于Python的Old-Style和New-Style Classes下次再探讨

**注意**：在Python3.x中不管是`Old-Style`还是`New-Style` Classes都是不允许对类的`__dict__`进行修改。因为不管哪种继承方式，它们都是`mappingproxy`对象

```python
>>> class O: pass
...
>>> O.__dict__['name'] = 'Finger'
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: 'mappingproxy' object does not support item assignment
>>> class O(object): pass
...
>>> O.__dict__['name'] = 'Finger'
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: 'mappingproxy' object does not support item assignment
```



