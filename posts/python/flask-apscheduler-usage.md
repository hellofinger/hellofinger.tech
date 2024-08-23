---
title: Flask-APScheduler 笔记
tags:
  - Python
  - APScheduler
  - Flask
  - Flask-APScheduler
categories:
  - Python
author: Finger
date: 2019-03-23 23:14:00
---

### 简介

Flask-APScheduler是Flask扩展，增加了对APScheduler的支持。

- 支持从Flask配置加载调度程序配置以及任务定义。
- 支持WEB程序和调度程序共存。提供REST API管理任务调度。并为REST API提供身份验证。目前只支持Basic-Auth认证。

官方地址：https://github.com/viniciuschiele/flask-apscheduler


### 使用
```
from flask import Flask
from flask_apscheduler import APScheduler


class Config(object):
    JOBS = [
        {
            'id': 'job1',
            'func': 'jobs:job1',
            'args': (1, 2),
            'trigger': 'interval',
            'seconds': 10
        }
    ]

    SCHEDULER_API_ENABLED = True


def job1(a, b):
    print(str(a) + ' ' + str(b))


if __name__ == '__main__':
    app = Flask(__name__)
    app.config.from_object(Config())

    scheduler = APScheduler()
    # it is also possible to enable the API directly
    # scheduler.api_enabled = True
    scheduler.init_app(app)
    scheduler.start()

    app.run()
```

### Flask Config 属性配置

```
SCHEDULER_JOBSTORES         # 配置存储任务存储器
SCHEDULER_EXECUTORS         # 配置执行器
SCHEDULER_JOB_DEFAULTS      # 配置调度默认属性
SCHEDULER_TIMEZONE          # 配置时区
SCHEDULER_AUTH              # 配置认证中心
SCHEDULER_API_ENABLED       # 配置是否开启API
SCHEDULER_API_PREFIX        # 配置API路由前缀
SCHEDULER_ENDPOINT_PREFIX   # 配置API路由后缀
SCHEDULER_ALLOWED_HOSTS     # 配置访问白名单
```

### REST API 接口
```
"""
Add the routes for the scheduler API.
"""
self._add_url_route('get_scheduler_info', '', api.get_scheduler_info, 'GET')
self._add_url_route('add_job', '/jobs', api.add_job, 'POST')
self._add_url_route('get_job', '/jobs/<job_id>', api.get_job, 'GET')
self._add_url_route('get_jobs', '/jobs', api.get_jobs, 'GET')
self._add_url_route('delete_job', '/jobs/<job_id>', api.delete_job, 'DELETE')
self._add_url_route('update_job', '/jobs/<job_id>', api.update_job, 'PATCH')
self._add_url_route('pause_job', '/jobs/<job_id>/pause', api.pause_job, 'POST')
self._add_url_route('resume_job', '/jobs/<job_id>/resume', api.resume_job, 'POST')
self._add_url_route('run_job', '/jobs/<job_id>/run', api.run_job, 'POST')
```

访问地址为：http://localhost:5000/scheduler/jobs

如果配置了 `SCHEDULER_API_PREFIX = "/api/v1/scheduler"`，则访问地址为: http://localhost:5000/api/v1/scheduler/jobs

### 配置API认证
```
SCHEDULER_AUTH = HTTPBasicAuth()

@scheduler.authenticate
def authenticate(auth):
    return auth['username'] == 'guest' and auth['password'] == 'guest'
```

完整示例：

```

from flask import Flask
from flask_apscheduler import APScheduler
from flask_apscheduler.auth import HTTPBasicAuth


class Config(object):
    JOBS = [
        {
            'id': 'job1',
            'func': '__main__:job1',
            'args': (1, 2),
            'trigger': 'interval',
            'seconds': 10
        }
    ]

    SCHEDULER_API_ENABLED = True
    SCHEDULER_AUTH = HTTPBasicAuth()


def job1(a, b):
    print(str(a) + ' ' + str(b))


if __name__ == '__main__':
    app = Flask(__name__)
    app.config.from_object(Config())

    scheduler = APScheduler()
    # it is also possible to set the authentication directly
    # scheduler.auth = HTTPBasicAuth()
    scheduler.init_app(app)
    scheduler.start()

    @scheduler.authenticate
    def authenticate(auth):
        return auth['username'] == 'guest' and auth['password'] == 'guest'

    app.run()
```

### 使用过程遇到的问题

#### Debug启动过程中重复运行的问题
```
# Bug: APScheduler in Flask executes twice
# https://github.com/viniciuschiele/flask-apscheduler/issues/58

app.run(debug=True, use_reloader=False)

# or 

manager.add_command("runserver", Server(use_debugger=False, use_reloader=False))
```

#### 多进程中APScheduler重复运行的问题
```
# Fix Multiple instances of scheduler problem
# https://github.com/viniciuschiele/flask-apscheduler/issues/51
import platform
import atexit
if platform.system() != 'Windows':
    fcntl = __import__("fcntl")
    f = open("scheduler.lock", "wb")
    try:
        fcntl.flock(f, fcntl.LOCK_EX | fcntl.LOCK_NB)
        scheduler.start()
        logger.debug("Scheduler Started...")
    except Exception as e:
        logger.error('Exit the scheduler, error - {}'.format(e))
        scheduler.shutdown()

    def unlock():
        fcntl.flock(f, fcntl.LOCK_UN)
        f.close()

    atexit.register(unlock)
else:
    msvcrt = __import__("msvcrt")
    f = open("scheduler.lock", "wb")
    try:
        msvcrt.locking(f.fileno(), msvcrt.LK_NBLCK, 1)
        scheduler.start()
        logger.debug("Scheduler Started...")
    except Exception as e:
        logger.error('Exit the scheduler, error - {}'.format(e))
        scheduler.shutdown()

    def _unlock_file():
        try:
            f.seek(0)
            msvcrt.locking(f.fileno(), msvcrt.LK_UNLCK, 1)
        except IOError:
            raise
    atexit.register(_unlock_file)
```

### 扩展
Flask-APScheduler缺少可视化任务调度和监控界面，需要使用者自己根据REST API扩展实现。


### 参考

- https://stackoverflow.com/questions/25504149/why-does-running-the-flask-dev-server-run-itself-twice
- https://blog.csdn.net/Raptor/article/details/69218271
