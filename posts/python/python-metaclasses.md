---
title: Python 元类(Metaclasses)
author: Finger
tags:
  - Python
categories:
  - Python
date: 2018-05-19 14:14:00
---

### Python 元类(Metaclasses)

#### 1、Type and Class

在Python3中，所有类都是新式类(New-Style)。因此，类的类型和它的实例类型是可以互换的。在Python中，所有的东西都是object。类也是一个object。因此类一定有一个type。
例如：

```python
>>> class Foo: pass
...
>>> x = Foo()
>>> type(x)
<class '__main__.Foo'>
>>> type(Foo)
<class 'type'>
```
如上所示：x的类型是Foo，但是Foo的类型，类的本身就是type。
我们熟悉的内置类的类型也是type:

```python
>>> for t in int, float, dict, list, tuple:
...     print(type(t))
...
<class 'type'>
<class 'type'>
<class 'type'>
<class 'type'>
<class 'type'>
```

type本身的类型也是type

```python
>>> type(type)
<class 'type'>
>>>
```

type是一个元类，其中类是实例。就像普通对象是类的实例，Python中的任何新式类，以及Python 3中的任何类，都是类的元类实例。

还是上述的例子：

```python
>>> class Foo: pass
...
>>> x = Foo()
>>> type(x)
<class '__main__.Foo'>
>>> type(Foo)
<class 'type'>
```

- x是类Foo的实例
- Foo 是type元类的实例
- type也是元类的一个实例，因此它是它自己的一个实例。


#### 2、如何动态创建类

内置类型()函数在传递一个参数时，返回对象的类型。对于新式类，通常与对象的__class__属性相同:

```python
>>> type(3)
<class 'int'>

>>> type(['foo', 'bar', 'baz'])
<class 'list'>

>>> t = (1, 2, 3, 4, 5)
>>> type(t)
<class 'tuple'>

>>> class Foo:
...     pass
...
>>> type(Foo())
<class '__main__.Foo'>
```

我们还可以使用三个参数的type(<name>， <base >， <dct>)方法来动态创建一个新类:

- <name> 指定了类名。这成为该类的__name__属性。
- <base> 指定了类继承的基类的一个元组。这成为该类的__bases__属性。
- <dct> 指定包含类主体定义的名称空间词典。这成为该类的__dict__属性。
以这种方式调用类型()将创建类型元类的一个新实例。换句话说，它动态创建一个新类。

##### Example 1 最基本的使用

```python
>>> Foo = type('Foo', (), {})
>>> x = Foo()
>>> x
<__main__.Foo object at 0x102b563c8>
```

上述通过type()动态创建类和下面代码创建类是一样的。

```python
>>> class Foo:
...     pass
...
>>> x = Foo()
>>> x
<__main__.Foo object at 0x102b56400>
```

最佳实战：动态创建类有一个很大好处。比如当你数据库表是采用分表设计。
如：

```python
table_1
table_2
table_3
...
table_N
```
这个时候你要根据表来创建对应的模型。你会发现table结构是一致，只是表名不同。
那么最好的方式就是通过type()元类动态创建模型type('table_'+id, (), {})。

##### Example 2 继承

```python
>>> Bar = type('Bar', (Foo,), dict(attr=100))

>>> x = Bar()
>>> x.attr
100
>>> x.__class__
<class '__main__.Bar'>
>>> x.__class__.__bases__
(<class '__main__.Foo'>,)
```

```python
>>> class Bar(Foo):
...     attr = 100
...

>>> x = Bar()
>>> x.attr
100
>>> x.__class__
<class '__main__.Bar'>
>>> x.__class__.__bases__
(<class '__main__.Foo'>,)
```

##### Example 3 默认属性

```python
>>> Foo = type(
...     'Foo',
...     (),
...     {
...         'attr': 100,
...         'attr_val': lambda x : x.attr
...     }
... )

>>> x = Foo()
>>> x.attr
100
>>> x.attr_val()
100
```

```python
>>> class Foo:
...     attr = 100
...     def attr_val(self):
...         return self.attr
...

>>> x = Foo()
>>> x.attr
100
>>> x.attr_val()
100
```

#### Examle 4 默认属性引用外部函数

```python
>>> def f(obj):
...     print('attr =', obj.attr)
...
>>> Foo = type(
...     'Foo',
...     (),
...     {
...         'attr': 100,
...         'attr_val': f
...     }
... )

>>> x = Foo()
>>> x.attr
100
>>> x.attr_val()
attr = 100
```

```python
>>> def f(obj):
...     print('attr =', obj.attr)
...
>>> class Foo:
...     attr = 100
...     attr_val = f
...

>>> x = Foo()
>>> x.attr
100
>>> x.attr_val()
attr = 100
```


### 参考文献

1、https://realpython.com/python-metaclasses/#type-and-class