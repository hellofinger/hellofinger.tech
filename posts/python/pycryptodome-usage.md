---
title: 使用PyCryptodome替代pyscrypto 
tags:
  - Python
  - pycrypto
categories:
  - Python
author: Finger
date: 2019-07-04 07:11:00
---

pycrypto 在Windows系统上安装比较麻烦，会出现意想不到的各种问题。因为pycrypto-2.6.1 之后就不在维护。因此系统兼容性就不那么友好。推荐PyCryptodome作为替代工具包。官方地址为：https://www.pycryptodome.org/en/latest/

```
# 1、Windows pip install pycryptodomex
# 2、Linux/Mac pip install pycryptodome
```

举个例子：

```
#!usr/bin/env python3
# -*- coding:utf-8 _*-

import base64
import hashlib
import json
import platform

if platform.system() != 'Windows':
    from Crypto.Cipher import AES
    from Crypto import Random
else:
    from Cryptodome.Cipher import AES
    from Cryptodome import Random
    
class AESCipher(object):
    def __init__(self, key):
        self.bs = 32
        self.key = hashlib.sha256(key.encode()).digest()

    def encrypt(self, raw):
        raw = self._pad(raw).encode('utf-8')
        iv = Random.new().read(AES.block_size)
        cipher = AES.new(self.key, AES.MODE_CBC, iv)
        return base64.b64encode(iv + cipher.encrypt(raw))

    def _pad(self, s):
        return s + (self.bs - len(s) % self.bs) * chr(self.bs - len(s) % self.bs)


def aes_encrypt(aes_key, app_client_id, app_client_secret):
    aes = AESCipher(aes_key)
    raw = dict(user=app_client_id, password=app_client_secret)
    data = aes.encrypt(json.dumps(raw))
    return bytes.decode(data)
```