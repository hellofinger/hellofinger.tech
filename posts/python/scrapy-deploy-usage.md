---
title: Scrapy 部署笔记
tags:
  - Scrapy
  - Scrapyd
  - SpiderKeeper
categories:
  - Python
author: Finger
date: 2017-07-02 10:41:00
---

关键字：

**scrapy**， **scrapyd**， **scrapy-client**， **spiderkeeper**

#### 一、新建一个scrapy项目

```
$ scrapy startproject scrapy_example
$ cd scrapy_example

# 修改配置文件scrapy.cfg
[settings]
default = scrapy_example.settings

[deploy:scrapyd1]
url = http://localhost:6800/
project = scrapy_example

```

#### 二、安装并启动scrapyd
```
$ pip install scrapyd

# 启动, 默认的端口是6800。可以在浏览器中查看结果，比如：http://127.0.0.1:6800/。
$ scrapyd

```

#### 三、发布工程到scrapyd
```
# 发布scrapyd需要安装scrapy-client
$ pip install scrapy-client
# 发布命令：scrapyd-deploy <target> -p <project>
$ scrapyd-deploy scrapyd1 -p scrapy_example

# 检查发布是否成功
$ scrapyd-deploy -l 或者 scrapyd-deploy -L scrapyd1

# 启动爬虫
$ curl http://127.0.0.1:6800/schedule.json -d project=scrapy_example -d spider=example
```

#### 四、使用spiderkeeper发布爬虫
```
# 生成egg文件
$ scrapyd-deploy --build-egg scrapy_example.egg

# 安装spiderkeeper(https://github.com/DormyMo/SpiderKeeper)
# pip install spiderkeeper
# 启动spiderkeeper
$ spiderkeeper --server=http://localhost:6800

# 访问 http://localhost:5000
# 这个时候可以在spiderkepper上传scrapy_example.egg文件，通过Web UI启动，监控爬虫
```

#### 参考文献

http://www.cnblogs.com/jinhaolin/p/5033733.html