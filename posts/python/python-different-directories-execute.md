---
title: Python不同目录下的调用情况
author: Finger
tags:
  - Python
categories:
  - Python
date: 2017-01-02 10:40:00
---

Python包含子目录中的模块方法比较简单，关键是能够在sys.path里面找到通向模块文件的路径。

下面将具体介绍几种常用情况:

（1）主程序与模块程序在同一目录下:
如下面程序结构:

```
`-- src
    |-- mod1.py
    `-- test1.py
```

若在程序test1.py中导入模块mod1, 则直接使用import mod1或from mod1 import *;

（2）主程序所在目录是模块所在目录的父(或祖辈)目录
如下面程序结构:

```
`-- src
    |-- mod1.py
    |-- mod2
    |   `-- mod2.py
    `-- test1.py
```

若在程序test1.py中导入模块mod2, 需要在mod2文件夹中建立空文件__init__.py文件(也可以在该文件中自定义输出模块接口); 
    然后使用 from mod2.mod2 import * 或import mod2.mod2.

（3）主程序导入上层目录中模块或其他目录(平级)下的模块
如下面程序结构:

```
`-- src
    |-- mod1.py
    |-- mod2
    |   `-- mod2.py
    |-- sub
    |   `-- test2.py
    `-- test1.py
```

若在程序test2.py中导入模块mod1和mod2。首先需要在mod2下建立__init__.py文件(同(2))，src下不必建立该文件。然后调用方式如下:
下面程序执行方式均在程序文件所在目录下执行，如test2.py是在cd sub;
之后执行python test2.py
而test1.py是在cd src;之后执行python test1.py; 不保证在src目录下执行python sub/test2.py成功。

```
import sys
sys.path.append("..")
import mod1
import mod2.mod2
```

（4）从(3)可以看出，导入模块关键是能够根据sys.path环境变量的值，找到具体模块的路径。这里仅介绍上面三种简单情况