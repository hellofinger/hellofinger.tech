---
title: Docker 构建第一个App
tags:
  - Flask
  - Docker
  - Ubuntu
categories:
  - Python
author: Finger
date: 2017-06-22 23:45:00
---



测试环境：阿里云 ubuntu14.04 


#### 一、新建Dockerfile文件

``` 
# Use an official Python runtime as a base image
FROM python:2.7-slim

# Set the working directory to /app
WORKDIR /app

# Copy the current directory contents into the container at /app
ADD . /app

# Install any needed packages specified in requirements.txt
RUN pip install -r requirements.txt

# Make port 80 available to the world outside this container
EXPOSE 80

# Define environment variable
ENV NAME World

# Run app.py when the container launches
CMD ["python", "app.py"]
```

#### 二、新建requirements.txt

```
Flask
Redis
```

#### 三、新建app.py

```
from flask import Flask
from redis import Redis, RedisError
import os
import socket

# Connect to Redis
redis = Redis(host="redis", db=0, socket_connect_timeout=2, socket_timeout=2)

app = Flask(__name__)

@app.route("/")
def hello():
    try:
        visits = redis.incr("counter")
    except RedisError:
        visits = "<i>cannot connect to Redis, counter disabled</i>"

    html = "<h3>Hello {name}!</h3>" \
           "<b>Hostname:</b> {hostname}<br/>" \
           "<b>Visits:</b> {visits}"
    return html.format(name=os.getenv("NAME", "world"), hostname=socket.gethostname(), visits=visits)

if __name__ == "__main__":
	app.run(host='0.0.0.0', port=80)

```

#### 四、编译

>ls  

>docker build -t friendlyhello . (当前目录)

>docker images


#### 五、运行

>docker run -p 4000:80 friendlyhello


#### 六、测试

>curl http://localhost:4000


#### 参考文献

https://docs.docker.com/get-started/part2/#run-the-app

