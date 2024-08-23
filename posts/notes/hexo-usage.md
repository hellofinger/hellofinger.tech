---
title: Hexo使用笔记
author: Finger
tags:
  - Hexo
categories:
  - Note
date: 2017-01-27 20:40:00
---

#### 中文文档：
https://hexo.io/zh-cn/docs/index.html

**参考博客：**
http://blog.csdn.net/poem_of_sunshine/article/details/29369785/
http://www.jianshu.com/p/465830080ea9#
http://www.jianshu.com/p/a2023a601ceb

**Github配置：**
https://help.github.com/articles/generating-an-ssh-key/
https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/


#### 常用命令
```
hexo server | s --debug
hexo clean
hexo generate | g
hexo deploy
```

绑定阿里云域名：
http://quantumman.me/blog/setting-up-a-domain-with-gitHub-pages.html

#### 后端管理插件hexo-admin
```
npm install --save hexo-admin
hexo server -d
open http://localhost:4000/admin/
```

**设置后台密码**
修改站点配置文件，就是网站根目录下的 _config.yml文件:
```
admin:
  username: finger
  password_hash: $2a$1$Cof3VuvY8nKKIUjeCNBSE.HjcrCKQ1P80GEegP//SLDFWZoGzm4pa
  secret: a secret something
```

- username：后端登录用户名
- password_hash：后端登录用户密码对应的md5 hash值
- secret：用于保证cookie安全

**密码生成**
hexo-admin密码是bcrypt编码。因此需要安装bcrypt-nodejs模块
```
$ node
> const bcrypt = require('bcrypt-nodejs')
> bcrypt.hashSync('your password secret here!')
//=> '$2a$10$8f0CO288aEgpb0BQk0mAEOIDwPS.s6nl703xL6PLTVzM.758x8xsi'
```
**在线生成**
https://www.bcrypt-generator.com/

#### Hexo主题：
- https://hexo.io/themes/
- https://github.com/henryhuang/oishi
- https://github.com/haojen/hexo-theme-Anisina
- https://github.com/stiekel/hexo-theme-random
- https://github.com/SuperKieran/TKL
- https://github.com/iissnan/hexo-theme-next


#### Next 皮肤设置
http://theme-next.iissnan.com/theme-settings.html