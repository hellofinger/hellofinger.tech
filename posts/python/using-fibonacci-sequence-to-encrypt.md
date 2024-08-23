---
title: 使用斐波拉契数列来做加密解密
tags:
  - Python
categories:
  - Python
author: Finger
date: 2018-07-14 22:23:00
---

#### 一、斐波那契数列定义

斐波那契数列（Fibonacci sequence），又称黄金分割数列、因数学家列昂纳多·斐波那契（Leonardoda Fibonacci）以兔子繁殖为例子而引入，故又称为“兔子数列”，指的是这样一个数列：1、1、2、3、5、8、13、21、34、……

数学表达式：
F(1)=1，F(2)=1, F(n)=F(n-1)+F(n-2)（n>=2，n∈N*）

```python
fib = lambda n: n if n <= 2 else fib(n - 1) + fib(n - 2)
```

#### 二、异或
运算规则：如果a、b两个值不相同，则异或结果为1。如果a、b两个值相同，异或结果为0。
特性：同一变量与另一变量和其异或值异或等于另一个数，如（a ^ b）^ b = a。

```python
>>> a = 2
>>> b = 3
>>> c = a ^ b
>>> c ^ b
2
```

#### 三、加密解密逻辑

- 1、定义斐波那契数列，用来做密码表。
- 2、将秘钥字符串与斐波拉契数列做关联，进行加密计算得到新的密码表。
- 3、加密，需要加密的原始字符串与加密计算后的新密码表进行异或，得到加密之后的字符串。
- 4、解密，把加密后的字符串与加密计算后的新密码表再进行异或就得到原始字符串


#### 四、代码实现

```python
# [1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89, 144...]
_fib = [1] * 64

# 初始化斐波拉契数列，本例定义数列大小为64
def __init_fib():
    for i in range(2, 64):
        _fib[i] = _fib[i-1] + _fib[i-2]
__init_fib()

def encrypt(source, secret):
    if not source or not secret:
        return source

    # 将字符串转成对应字符的ascii码数值
    src_list = list(map(ord, source))
    key_list = list(map(ord, secret))
    key_len = len(key_list)

    # 32 <= N <= 64
    N = max(sum(key_list) & 0x3F, 32)
    # 将秘钥字符串与斐波拉契数列做加密计算，生成新的列表
    # 保证计算后的数值小于255
    tmp = [(key_list[_fib[i] % key_len] + _fib[i]) & 0xFF for i in range(N)]
    # 将原始字符串与新生成的列表做异或
    dst_list = [src_list[i] ^ tmp[i % N] for i in range(len(src_list))]
    # chr 将范围[0-255]之间的数字转换成ascii码字符
    return ''.join(list(map(chr, dst_list)))


# 对称加密，加密即是解密
decrypt = encrypt


if __name__ == '__main__':
    secret_key = 'have a nice day'
    dst_string = encrypt('FingerChou', secret_key)
    print(dst_string)
    print(decrypt(dst_string, secret_key))
 
```

#### 五、综述

上述只是利用斐波拉契数列进行了简单的加密计算。如果知道数列也就知道密码了。
虽然我们还增加了秘钥字符串。增加了复杂性。但是实际应用中，还是要把生成密码表的算法设计的更复杂一点。







