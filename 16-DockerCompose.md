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

```yaml
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

### 1、下载

```shell
curl -L "https://github.com/docker/compose/releases/download/v1.26.2/docker-compose-$(uname-s)-$(uname -m)" -o /usr/local/bin/docker-compose

// 推荐
curl -L "https://get.daocloud.io/docker/compose/releases/download/v1.25.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

curl -SL https://get.daocloud.io/docker/compose/releases/download/v2.7.0/docker-compose-linux-x86_64 -o /usr/local/bin/docker-compose
```

#### 1) uname 说明

> ​	以上，`uname`为查看*内核 / 操作系统 / CPU 信息命令。*

| 选项                    | 说明                                                         |
| ----------------------- | ------------------------------------------------------------ |
| -a, --all               | 以如下次序输出所有信息。其中若-p 和 -i 的探测结果不可知则被省略： |
| -s, --kernel-name       | 输出内核名称                                                 |
| -n, --nodename          | 输出网络节点上的主机名                                       |
| -r, --kernel-release    | 输出内核发行号                                               |
| -v, --kernel-version    | 输出内核版本                                                 |
| -m, --machine           | 输出主机的硬件架构名称                                       |
| -p, --processor         | 输出处理器类型或"unknown"                                    |
| -i, --hardware-platform | 输出硬件平台或"unknown"                                      |
| -o, --operating-system  | 输出操作系统名称                                             |

#### 2) pip 说明	

> ​	若使用 pip install docker-compose 命令安装 compose
>
> ​	需先安装 epel-release 和 python-pip
>

### 2、设置文件可执行权限

```shell
chmod +x /usr/local/bin/docker-compose
```

### 3、查看版本信息
```shell
docker-compose version
```

## Getting Started

### Step 1: Setup

#### 1) 创建一个 目录

```shell
mkdir composetest
cd composetest
```

#### 2) 创建一个 app.py

> ​	以下中，`redis`是应用程序网络上的 redis 容器的主机名。我们使用 Redis 的默认端口，`6379`。

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

#### 3) 创建一个 requirements.txt

```shell
flask
redis
```

### Step 2: 创建 Dockerfile

```dockerfile
# syntax=docker/dockerfile:1

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
```

#### 0) 说明

- 从 Python 3.7 映像开始构建映像。
- 将工作目录设置为`/code`。
- 设置命令使用的环境变量`flask`。
- 安装 gcc 和其他依赖项。
- 复制`requirements.txt`并安装 Python 依赖项。
- 向镜像添加元数据以描述容器正在侦听端口 5000。
- 将项目中的当前目录复制`.`到镜像中的workdir`.`。
- 将容器的默认命令设置为`flask run`。

### Step 3: 在 docker-compose.yml 中定义服务

#### 1) 创建 docker-compose.yml

> ​	该 docker-compose.yml 定义了 web 和 redis 服务。
>
> ​	web service:
> ​		该 web service 从当前目录中的`Dockerfile`构建镜像。然后将容器和主机绑定到暴露的端口`8000`。 该 web service 使用 Flask Web 服务器的默认端口`5000`。 
>
> ​	redis service:
> ​		该 redis service 使用的是 DockerHub 中的开源镜像。

```yaml
version: "3.0"
services:
  web:
    build: .
    ports:
      - "8000:5000"
  redis:
    image: "redis:alpine"
```

### Step 4: 启动 compose 项目
```shell
docker compose up -d  # 后台运行
```

