[toc]

# Docker Compose

## Docker Compose 概述

> ​	Docker Compose 是一个<u>*用于定义和运行*</u>多容器 Docker 应用程序的工具(批量容器编排)。
>
> ​	使用 Docker Compose，可以<u>*使用 YAML 文件*</u>来配置应用程序的服务。然后，<u>*使用一个命令*</u>，即可从配置中创建并启动所有服务。
>
> ​	Docker Compose 适用于所有环境：生产、登台、开发、测试以及 CI 工作流程。
>
> ​	通过 Docker Compose 可以启动我们的项目(Project)，即我们的一组相关联的容器。如：blog、web、mysql。

## Docker Compose 使用步骤

1. 用`Dockerfile`定义应用环境，以便它能够在任何地方被使用。

2. 在`docker-compose.yml`中定义组成应用程序的服务，这样它们就可以在一个隔离的环境中一起运行。

3. 运行`docker compose up`命令，Docker Compose 命令将会启动并运行所有应用程序。也可以使用`Compose Standalone(docker-compose binary)`运行`docker-compose up`。

### docker-compose.yml 样例

```shell
version: "3.9"  # optional since v1.27.0
services:	# 即我们的容器、应用等。如：MySQL、Redis、Web等。
  web:
    build: .
    ports:
      - "8000:5000"
    volumes:
      - .:/code
      - logvolume01:/var/log
    depends_on:  # 以前版本中的 links
      - redis
  redis:
    image: redis
volumes:
  logvolume01: {}
```

## Docker Compose 安装

### 安装

```shell
curl -L "https://get.daocloud.io/docker/compose/releases/download/v1.25.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

pip install docker-compose

2、设置文件可执行权限
chmod +x /usr/local/bin/docker-compose

3、查看版本信息
docker-compose -version

4、创建composetest文件夹
mkdir composetest
cd composetest

#制作程序

vim app.py

```python
import time

import redis
from flask import Flask

app = Flask(__name__)
cache = redis.Redis(host='redis', port=6379)

def get_hit_count():
    retries = 5
    while True:
        try:
            return cache.incr('hits')
        except redis.exceptions.ConnectionError as exc:
            if retries == 0:
                raise exc
            retries -= 1
            time.sleep(0.5)

@app.route('/')
def hello():
    count = get_hit_count()

	return 'Hello World! I have been seen {} times.\n'.format(count)
```

#依赖包

vim requirements.txt

flask

redis

#构建镜像文件

vim Dockerfile

syntax=docker/dockerfile:1

RUN sed -i 's/https/http/' /etc/apk/repositories
RUN apk add curl

FROM python:3.7-alpine
WORKDIR /code
ENV FLASK_APP=app.py
ENV FLASK_RUN_HOST=0.0.0.0
RUN apk add --no-cache gcc musl-dev linux-headers
COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt
EXPOSE 5000
COPY . .

CMD ["flask", "run"]

#部署应用

vim docker-compose.yml

version: "3"
services:
  web:
    build: .
    ports:

    - "8000:5000"
  redis:

    image: "redis:alpine"

#启动compose项目
docker-compose up
