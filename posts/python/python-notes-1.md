---
title: Python 杂记(一)
tags:
  - Python
categories:
  - Python
author: Finger
date: 2019-07-11 07:31:00
---

### 使用 __slots__

`__slots__`变量是用来限制class能添加的属性

例如
```
>>> class Student(object):
...     __slots__ = ('name', 'age') # 用tuple定义允许绑定的属性名称
...
```

```
>>> s = Student() # 创建新的实例
>>> s.name = 'Michael' # 绑定属性'name'
>>> s.age = 25 # 绑定属性'age'
>>> s.score = 99 # 绑定属性'score'
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: 'Student' object has no attribute 'score'
```
从上面代码示例我们可以看到由于`score`没有被放到`__slots__`中，所以不能绑定`score`属性，试图绑定`score`将得到AttributeError的错误。
使用`__slots__`要注意，`__slots__`定义的属性仅对当前类起作用，对继承的子类是不起作用的：
```
>>> class GraduateStudent(Student):
...     pass
...
>>> g = GraduateStudent()
>>> g.score = 9999
```
除非在子类中也定义`__slots__`，这样，子类允许定义的属性就是自身的`__slots__`加上父类的`__slots__`。

详情可以查看廖雪峰教程：
https://www.liaoxuefeng.com/wiki/897692888725344/923030542875328


### 关于six.with_metaclass(ABCMeta, object)的理解

元类就是创建类的类，在Python中，只有type类及其子类才可以当元类，type创建类时，参数格式：type(classname, parentclasses,attrs), classname是类名，字符串类型，parentclasses是类的所有父类，元组类型，attrs是类的所有{属性:值}，是字典类型。一切类的创建最终都会调用type.__new__(cls, classname, bases, attrs)，它会在堆中创建一个类对象，并返回该对象。

当通过type.__new__(cls, classname, bases, attrs)创建类时，cls就是该类的元类，它是type类或其子类。


ABCMeta是一个抽象类的元类，用来创建抽象类基类：Metaclass for defining Abstract Base Classes (ABCs).
six.with_metaclass是用元类来创建类的方法，调用一个内部类，在内部类的__new__函数中，返回一个新创建的临时类。

```
def with_metaclass(meta, *bases):
    """Create a base class with a metaclass."""
    # This requires a bit of explanation: the basic idea is to make a dummy
    # metaclass for one level of class instantiation that replaces itself with
    # the actual metaclass.
    class metaclass(type):

        def __new__(cls, name, this_bases, d):
            return meta(name, bases, d)

        @classmethod
        def __prepare__(cls, name, this_bases):
            return meta.__prepare__(name, bases)
    return type.__new__(metaclass, ‘temporary_class‘, (), {})
```

six.with_metaclass(ABCMeta, object)就通过内部类的__new__函数返回一个ABCMeta元类创建的临时类，作为PeopleBase类的基类。

在python里类也是对象，下面两个语句可以看PeopleBase类的类型，都是<class 'abc.ABCMeta'>，即PeopleBase类是ABCMeta元类的对象，是由ABCMeta元类生成的。

```
class PeopleBase(six.with_metaclass(ABCMeta, object)):
    @abstractmethod
    def work(self, *args, **kwargs):
        #pass为空语句，占位用
        pass

    @abstractmethod
    def live(self, *args, **kwargs):
        pass

print(type(PeopleBase))
print(PeopleBase.__class__)
#<class ‘abc.ABCMeta‘>
#<class ‘abc.ABCMeta‘>
```

详情请查看：
- https://www.cnblogs.com/zhaoshizi/p/9180886.html
- https://segmentfault.com/a/1190000012530796

### python中MRO排序原理

#### 拓扑排序
![](/imgs/20190712/1.png)

如上图所示每个点都是有指向性的：可能指向别人或者被别人指向。

拓扑顺序就是：**每次找到一个只指向别人的点 (学术性说法：入度为0)，记录下来；然后忽略掉这个点和它所指出去的线，再找到下一个只指向别人的点，记录下来，直到剩最后一个点**，所有记录的点的顺序就是拓扑顺序

![](/imgs/20190712/2.png)

上图中，只有点1只指向别人，输出1；去掉点1和它伸出的两根线外只有点2只指向别人，输出2；...类推下去，得到拓扑排序结构: 1 2 4 3 5

#### MRO 排序算法
MRO 排序应用了 C3 算法，想了解 C3 自己查吧...总之得到的结果类似于拓扑排序，下面有段简单的多继承代码和其对应的拓扑排序的抽象图

```
class D(object):
    pass

class E(object):
    pass
 
class F(object):
    pass
 
class C(D, F):
    pass
 
class B(E, D):
    pass
 
class A(B, C):
    pass
 
if __name__ == '__main__':
    print A.__mro__
```

得到的输出结果：

```
(<class '__main__.A'>, <class '__main__.B'>, <class '__main__.E'>, <class '__main__.C'>, <class '__main__.D'>, <class '__main__.F'>, <type 'object'>)
```

下面就是抽象出来的图

![](/imgs/20190712/3.jpg)

我们就用拓扑排序来分析，但是这里会碰到同时存在好几个点都是入度为0 (说人话，就是没有被别人指向的点)，这时按照树的排序来，即从左到右，从根到叶，这里 A 就是根。
所以具体情况就是：我们先找到了点 A只有它没有被别人指向，输出A；去掉A及其伸出的两条线，剩 B 和 C 点同时满足只指向别人，按照树的顺序从左到右，故先输出B；去掉线剩 E 和 C ，输出E ;去线剩 C，输出C；去线剩 D 和 F ，输出D；去线只剩F ，输出F；最后输出object;得到的输出结果：

```
A B E C D F object
```

原文地址：
https://www.jianshu.com/p/6651ed35104c