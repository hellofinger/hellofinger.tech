---
title: Python3.6中那些很酷的特性
author: Finger
tags:
  - Python
categories:
  - Python
date: 2018-05-22 20:40:00
---


#### 可读性更强的数字字面值

Python代码在可读性上做到了极致，被称为是可执行伪代码。然而，它还在不断地改进，比如这个可读性更好的数字字面值语法，就是方便程序员能以一种 “for humans ” 的方式阅读和理解数字。你现在可以给数字添加下划线，并按照你喜欢的方式对它们进行分组  。这对于二进制或十六进制数字或者是大数来说非常方便：

```
>>> six_figures  = 100_000
>>> six_figures
100000

>>> a = 10_00_0
>>> a
10000

>>> error  = 0xbad_c0ffee
>>> error
50159747054

>>> flags  = 0b_0111_0101_0001_0101
>>> flags
29973
```
请记住，这种改变只是语法层面上的变化，是一种在源代码中以不同方式表示数字文字的方法而已。在虚拟机编译成字节码的时候不会有任何变化，你可以在 PEP515 中了解到关于它的更多信息。

#### 新的字符串格式化方法

对字符串格式化操作有两种常用的方法，第一个是使用 “%” 操作符，第二个是使用 `format` 函数。

**“%” 操作符**

```
>>> s = "%s is %d" % ('two', 2)
>>> s
'two is 2'
```

**`format` 函数**

```
>>> s = "{fruit} is {color}".format(fruit='apple', color='red')
>>> s
'apple is red'
```
显然，format 函数要比 % 操作符的可读性要好，在Python 3.6 增加了第三种格式化字符串方法，称为 **Formatted String Literals** ，简称 **f字符串**。

```
>>> name = 'Bob'
>>> f'Hello, {name}!'
'Hello, Bob!'
```
你还可以在字符串内使用嵌入式的 Python 表达式，例如：

```
>>> a = 5
>>> b = 10
>>> f'Five plus ten is {a + b} and not {2 * (a + b)}.'
'Five plus ten is 15 and not 30.'
```

这个看起来很酷，其实这种操作在模版引擎中早就有这样的特性存在，只不过因为用的人多了，就引入到了语言标准中。

除了这些，还可以操作数字

```
# 精度
>>> PI = 3.141592653
>>> f"Pi is {PI:.2f}"
>>> 'Pi is 3.14'

>>> error = 50159747054
#以16进制格式化
>>> f'Programmer Error: {error:#x}'
'Programmer Error: 0xbadc0ffee'

#以二进制格式化
>>> f'Programmer Error: {error:#b}'
'Programmer Error: 0b101110101101110000001111111111101110'
```

你可以在 PEP498 中了解更多信息

**变量注释**

“动态语言一时爽，代码重构火葬场”，虽有危言耸听嫌疑，但的确因为动态语言的灵活性也带来代码维护困难的麻烦，我们不得不通过文档注释来对参数进行说明，而有时又因为业务需求的变更导致代码修改后没有同步文档注释造成实际代码和文档不一致的情况，如果能像静态语言一样，让程序员在语法层面就是就被限制在规则范围内做事，就不会出问题了。所以，像Java这样的语言做工程项目是有优势的。

从Python 3.5开始，可以将类型注解添加到函数和方法中：

```
>>> def my_add(a: int, b: int) -> int:
...    return a + b
```

这个函数表示，a 和 b 两个参数必须是 int 类型，函数的返回值也是 int。

在语义方面没有任何改变—CPython解释器只是将类型记录为类型注释，但不做任何方式类型检查。类型检查纯粹是可选的，你需要一个像Mypy这样的工具来帮助你。

可以在PEP 526中了解更多关于这一变化的信息。

当然，这个版本不止这么一点点变化，还有：
- 异步生成器的语法
- 异步推导式语法
- 更快的字典结构，内存减少20％到25％

英文原文：https://dbader.org/blog/cool-new-features-in-python-3-6