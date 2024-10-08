---
title: Numpy基础
tags:
  - Python
  - Numpy
categories:
  - Python
author: Finger
date: 2018-08-23 22:19:00
---

原文地址：https://www.jiqizhixin.com/articles/2018-07-18-2

#### 介绍

numpy是专门为科学计算设计的一个python扩展包，为python提供高效率的多维数组，也被称为面向阵列计算（array oriented computing），同时numpy也是github上的一个开源项目：numpy，numpy是基于c语言开发，所以这使得numpy的运行速度很快，高效率运行就是numpy的一大优势。

用numpy生成一维数组
```
>>> import numpy as np
>>> a = np.array([[1,2,3]], dtype=np.int32)
>>> a
array([[1, 2, 3]])
```

numpy中定义的最重要的数据结构是称为ndarray的n维数组类型，这个结构引用了两个对象，一块用于**保存数据的存储区域**和一个**用于描述元素类型的dtype对象**： 

![](/imgs/20180823/20180516210240962.jpg)

#### 性能

二维数组的生成在python中我们还可以用到list列表，如果用list来表示[1,2,3]，由于list中的元素可以是任何对象，所以list中保存的是对象的指针，如果要保存[1,2,3]就需要三个指针和三个整数对象，是比较浪费内存资源和cpu计算时间的，而ndarray是一种保存单一数据类型的多维数组结构，在数据处理上比list列表要快上很多。

#### 使用

##### 生成一个3x3的矩阵
```
>>> import numpy as np
>>> data = np.array([[1,2,3],[4,5,6],[7,8,9]])
>>> data
array([[1, 2, 3],
       [4, 5, 6],
       [7, 8, 9]])

# 等价于data=np.arange(1，10).reshape(3,3)
>>> data = np.arange(1,10).reshape(3,3)
>>> data
array([[1, 2, 3],
       [4, 5, 6],
       [7, 8, 9]])
```

##### 查看矩阵信息
```
# 返回元组，表示n行n列
>>> data.shape
(3, 3)

# 返回数组数据类型
>>> data.dtype
dtype('int32')

# 返回是几维数组
>>> data.ndim
2
```

##### 转换数据类型
```
# 拷贝一份新的数组
>>> a = data.astype(float)
>>> a
array([[1., 2., 3.],
       [4., 5., 6.],
       [7., 8., 9.]])
>>> a.dtype
dtype('float64')
```

##### 数组之间的计算
```
>>> data + data
array([[ 2,  4,  6],
       [ 8, 10, 12],
       [14, 16, 18]])
>>> data * data
array([[ 1,  4,  9],
       [16, 25, 36],
       [49, 64, 81]])
```

可以看出相同规格的数组计算是直接作用在其元素级上的，那不同的规格的数组是否能进行运算呢，我们来看下这个例子：
```
# 生成一个2x2numpy数组
>>> data1 = np.array([[1,2],[1,2]])
>>> data + data1
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ValueError: operands could not be broadcast together with shapes (3,3) (2,2)
```
我们可以看出不同规格的数组一起计算的话是会报出**广播错误**的，那是不是可以下结论了，别急我们再来看下方两个特殊例子：
```
>>> data2 = np.array([[1,2,3]])
>>> data + data2
array([[ 2,  4,  6],
       [ 5,  7,  9],
       [ 8, 10, 12]])
>>> data3 = np.array([[1], [2], [3]])
>>> data + data3
array([[ 2,  3,  4],
       [ 6,  7,  8],
       [10, 11, 12]])
```

data2数组的列数量与data数组相等，data3数组的行数量与data数组相等，这两个numpy数组虽然规格与data数组不一样，但却依然可以与data数组进行运算。

##### 数据的切片
```
# 沿着行(axis=0)进行索引
>>> data[:2]
array([[1, 2, 3],
       [4, 5, 6]])

# 先沿着行(axis=0)进行索引，再沿着列(axis=1)进行索引
>>> data[:2, :2]
array([[1, 2],
       [4, 5]])

# 下标是从0开始
>>> data[1,0:2]
array([4, 5])
```

这里需要注意的是，切片操作是在原始数组上创建一个视图view，这只是访问数组数据的一种方式。 因此原始数组不会被复制到内存中，传递的是一个类似引用的东西，与上面的astype（）方法是两种不同的拷贝方式，这里我们来看一个例子：
```
>>> a = data[1]
>>> a
array([4, 5, 6])
>>> a[:] = 0
>>> data
array([[1, 2, 3],
       [0, 0, 0],
       [7, 8, 9]])
```

当切片对象a改变时，data的对应值也会跟着改变，这是在我们日常数据处理中有时会疏忽的一个点，最安全的复制方法是使用copy（）方法进行浅拷贝：
```
>>> a = data[1].copy()
>>> a
array([0, 0, 0])
>>> a[:] = 9
>>> data
array([[1, 2, 3],
       [0, 0, 0],
       [7, 8, 9]])
```

##### 数组的布尔索引
```
>>> data
array([[1, 2, 3],
       [4, 5, 6],
       [7, 8, 9]])
>>> data > 3
array([[False, False, False],
       [ True,  True,  True],
       [ True,  True,  True]])
# 找出大于3的元素
>>> data[data > 3]
array([4, 5, 6, 7, 8, 9])
```

##### 数组的逻辑表达处理
```
# 大于3的标记为1，小于等于3的标记为0
>>> np.where(data>3,1,0)
array([[0, 0, 0],
       [1, 1, 1],
       [1, 1, 1]])
```

##### 数组的常用统计操作
```
# 沿着行(axis=0)进行索引，求出其平均值
>>> data.mean(axis=0)
array([4., 5., 6.])
# 求出全部元素的方差
>>> data.std()
2.581988897471611
# 统计数组中元素大于3的个数
>>> (data > 3).sum()
6

# 数组中是否存在一个或多个true
>>> data.any()
True

# 数组中是否全部数都是true
>>> data.all()
True

# 沿着行(axis=0)进行索引，进行累加
>>> data.cumsum(0)
array([[ 1,  2,  3],
       [ 5,  7,  9],
       [12, 15, 18]], dtype=int32)

# 沿着列(axis=1)进行索引，进行累乘
>>> data.cumprod(1)
array([[  1,   2,   6],
       [  4,  20, 120],
       [  7,  56, 504]], dtype=int32)
```

##### 数组的排序操作
```
>>> data = np.random.randn(4,4)
>>> data
array([[-2.09150248, -0.07758111, -1.30427684, -0.79387749],
       [-0.10272387, -0.89728241, -0.54607326,  0.87435473],
       [ 1.83972061, -0.81256093, -1.21652558,  0.51171488],
       [-0.40912271,  0.21529186,  0.66593734, -0.54275447]])

# 沿着行(axis=0)进行索引，并进行升序排序
>>> data.sort(0)
>>> data
array([[-2.09150248, -0.89728241, -1.30427684, -0.79387749],
       [-0.40912271, -0.81256093, -1.21652558, -0.54275447],
       [-0.10272387, -0.07758111, -0.54607326,  0.51171488],
       [ 1.83972061,  0.21529186,  0.66593734,  0.87435473]])

# 降序操作
>>> data[::-1]
array([[ 1.83972061,  0.21529186,  0.66593734,  0.87435473],
       [-0.10272387, -0.07758111, -0.54607326,  0.51171488],
       [-0.40912271, -0.81256093, -1.21652558, -0.54275447],
       [-2.09150248, -0.89728241, -1.30427684, -0.79387749]])
```

**注意：直接调用数组的方法的排序将直接改变数组而不会产生新的拷贝。**

##### 矩阵运算
```
>>> x=np.arange(9).reshape(3,3)
>>> x
array([[0, 1, 2],
       [3, 4, 5],
       [6, 7, 8]])
# 矩阵相乘
>>> np.dot(x, x)
array([[ 15,  18,  21],
       [ 42,  54,  66],
       [ 69,  90, 111]])
# 矩阵转置
>>> x.T
array([[0, 3, 6],
       [1, 4, 7],
       [2, 5, 8]])
```

