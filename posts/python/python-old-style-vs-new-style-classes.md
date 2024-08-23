---
title: Python Old-Style Vs. New-Style Classes
author: Finger
tags:
  - Python
categories:
  - Python
date: 2018-05-18 21:40:00
---

### Python Old-Style Vs. New-Style Classes

在Python中，类有两种方式定义，一种是非正式的，我们称为旧式的或者叫经典的(Old-Style)，另一种是正式的(New-Style)。

换一种说法，对于继承自object的类，则称为New-Style类。未继承object的类称为Old-Style类。

在Python2.7中New-Style和Old-Style类存在一些区别。

#### Old-Style Classes
对于Old-Style类的实例，它的类和类型并不相同。Old-Style类的实例是一个内置类型，而类型是一个实例。例如：obj是一个Old-Style的实例。那么obj.__class__是一个内置类，而type(obj)是一个实例。


```python
>>> class Foo: pass
...
>>> obj = Foo()
>>> obj.__class__
<class __main__.Foo at 0x103faba10>
>>> type(obj)
<type 'instance'>
```

#### New-Style Classes
New-Style类统一了类和类型的概念。例如：obj是一个New-Style的实例，obj.__class__和type(obj)相同:

```python
>>> class Foo(object): pass
...
>>> obj = Foo()
>>> obj.__class__
<class '__main__.Foo'>
>>> type(obj)
<class '__main__.Foo'>
```

```
>>> n = 10
>>> d = {'x': 1, 'y': 2}
>>> x = Foo()
>>> for obj in (n, d, x):
...     print(type(obj) is obj.__class__)
...
True
True
True
```

在 Python 2.7中New-Style类和Old-Style类在多继承方面也会有差异：

#### Old-Style Classes

```python
class A:
    def foo(self):
        print('called A.foo()')

class B(A):
    pass

class C(A):
    def foo(self):
        print('called C.foo()')

class D(B, C): 
    pass

if __name__ == '__main__':
    d = D() 
    d.foo()

```

```
Out: called A.foo()
```


#### New-Style Classes

```python
class A(object):
    def foo(self):
        print('called A.foo()')

class B(A):
    pass

class C(A):
    def foo(self):
        print('called C.foo()')

class D(B, C): 
    pass

if __name__ == '__main__':
    d = D() 
    d.foo()

```

```
Out: called C.foo()
```

B、C 是 A 的子类，D 多继承了 B、C 两个类，其中 C 重写了 A 中的 foo() 方法。

如果 A 是经典类，当调用 D 的实例的 foo() 方法时，**Python 会按照深度优先的方法去搜索 foo()**，路径是 B-A-C ，执行的是 A 中的 foo() ；

如果 A 是新式类，当调用 D 的实例的 foo() 方法时，**Python 会按照广度优先的方法去搜索 foo()**，路径是 B-C-A ，执行的是 C 中的 foo() 。

因为 D 是直接继承 C 的，从逻辑上说，执行 C 中的 foo() 更加合理，因此新式类对多继承的处理更为合乎逻辑。

**注意：** 在 Python 3.x 中的新式类貌似已经兼容了经典类，无论 A 是否继承 object 类， D 实例中的 foo() 都会执行 C 中的 foo() 。但是在 Python 2.7 中这种差异仍然存在，因此还是推荐使用新式类，要继承 object 类。

#### 参考文献
1、https://realpython.com/python-metaclasses/#old-style-vs-new-style-classes

2、https://www.zhihu.com/question/19754936/answer/202650790

