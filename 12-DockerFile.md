[toc]

# DockerFile

> ​	DockerFile 是用来构建 Docker 镜像文件的。
>
> ​	DockerFile 是面向开发的。

DockerFile：用来构建镜像的文件。定义了一切步骤。就如源代码那样。

Docker Image：DockerFile 生成的镜像。是最终要发布和运行的产品。

Docker Container： 是用来提供服务的。

## DockerFile 构建步骤：

1. 编写一个 dockerfile 文件：

   ```shell
   vim /home/dockerfile-test/dockerfile
   ```

2. docker build 构建一个镜像：

   ```shell
   docker build -f /home/dockerfile-test/dockerfile -t centos:1.1 .
   ```

3. docker run 运行镜像：

   ```shell
   docker run -it --name centos02 centos:1.1
   ```

4. docker push 发布镜像：

## DockerFile 命令

- FROM- 基础镜像(基于哪个镜像来构建镜像)。

- MAINTAINER- 镜像维护者信息。(姓名+邮箱)。

- RUN- 构建镜像执行的命令，每一次 RUN 都会构建一层。

- CMD- 容器启动的命令。如果有多个则以最后一个为准(可被替代)，也可以为 ENTRYPOINT 提供参数。

- VOLUME- 定义数据卷，如果没有定义则使用默认。指定容器挂载点到宿主机自动生成的目录或其他容器。

- USER- 指定后续执行的用户和用户组。

- WORKDIR- 为 RUN、CMD、ENTRYPOINT、COPY 和 ADD 设置工作目录，就是当进入到容器中所切换进入的目录。

- HEALTHCHECH- 健康检测指令

- ARG- 变量属性值，但不在容器内部起作用

- EXPOSE- 指定对外的端口。

- ENV- 设置容器环境变量，容器内部也会起作用。

- ADD- 添加文件或目录到容器中，如果是URL或压缩包便会自动下载或自动解压。

- COPY- 添加文件，以复制的形式。

- ENTRYPOINT- 容器进入时执行的命令(可以追加命令)。
- ONBUILD- 用于延迟构建命令的执行。简单的说，就是 Dockerfile 里用 ONBUILD 指定的命令，在本次构建镜像的过程中不会执行（假设镜像为 test-build）。当有新的 Dockerfile 使用了之前构建的镜像 FROM test-build ，这时执行新镜像的 Dockerfile 构建时候，会执行 test-build 的 Dockerfile 里的 ONBUILD 指定的命令。

### CMD 与 ENTRYPOINT 的区别：

| 命令       | 说明                                                         |
| ---------- | ------------------------------------------------------------ |
| CMD        | 容器启动的命令。如果有多个则以最后一个为准(可被替代)，也可以为 ENTRYPOINT 提供参数。 |
| ENTRYPOINT | 容器进入时执行的命令(可以追加命令)。                         |

#### CMD

```shell
FROM centos:7
CMD ["ls","-l"]

[root@localhost dockerfile-test]# docker run cmdtest -l
docker: Error response from daemon: failed to create shim task: OCI runtime create failed: runc create failed: unable to start container process: exec: "-l": executable file not found in $PATH: unknown.
ERRO[0000] error waiting for container: context canceled 
```

#### ENTRYPOINT

```shell
FROM centos:7
ENTRYPOINT ["ls","-l"]

[root@localhost dockerfile-test]# docker run entrypointtest:1.0 -l
total 12
drwxr-xr-x.   1 root root     6 Aug 22 03:32 .
drwxr-xr-x.   1 root root     6 Aug 22 03:32 ..
-rwxr-xr-x.   1 root root     0 Aug 22 03:32 .dockerenv
-rw-r--r--.   1 root root 12114 Nov 13  2020 anaconda-post.log
lrwxrwxrwx.   1 root root     7 Nov 13  2020 bin -> usr/bin
drwxr-xr-x.   5 root root   340 Aug 22 03:32 dev
drwxr-xr-x.   1 root root    66 Aug 22 03:32 etc
drwxr-xr-x.   2 root root     6 Apr 11  2018 home
lrwxrwxrwx.   1 root root     7 Nov 13  2020 lib -> usr/lib
lrwxrwxrwx.   1 root root     9 Nov 13  2020 lib64 -> usr/lib64
drwxr-xr-x.   2 root root     6 Apr 11  2018 media
drwxr-xr-x.   2 root root     6 Apr 11  2018 mnt
drwxr-xr-x.   2 root root     6 Apr 11  2018 opt
dr-xr-xr-x. 311 root root     0 Aug 22 03:32 proc
dr-xr-x---.   2 root root   114 Nov 13  2020 root
drwxr-xr-x.  11 root root   148 Nov 13  2020 run
lrwxrwxrwx.   1 root root     8 Nov 13  2020 sbin -> usr/sbin
drwxr-xr-x.   2 root root     6 Apr 11  2018 srv
dr-xr-xr-x.  13 root root     0 Aug 22 02:24 sys
drwxrwxrwt.   7 root root   132 Nov 13  2020 tmp
drwxr-xr-x.  13 root root   155 Nov 13  2020 usr
drwxr-xr-x.  18 root root   238 Nov 13  2020 var

```

## 案例

```
FROM scratch
ADD centos-7-x86_64-docker.tar.xz /

LABEL \
    org.label-schema.schema-version="1.0" \
    org.label-schema.name="CentOS Base Image" \
    org.label-schema.vendor="CentOS" \
    org.label-schema.license="GPLv2" \
    org.label-schema.build-date="20201113" \
    org.opencontainers.image.title="CentOS Base Image" \
    org.opencontainers.image.vendor="CentOS" \
    org.opencontainers.image.licenses="GPL-2.0-only" \
    org.opencontainers.image.created="2020-11-13 00:00:00+00:00"

CMD ["/bin/bash"]
```

### 构建自己的镜像

#### /home/dockerfile-test/dockerfile

```shell
FROM centos:7
MAINTAINER nishi<123@163.com>

ENV MYPATH /usr/local

WORKDIR $MYPATH

RUN yum install -y vim
RUN yum install -y net-tools

EXPOSE 80

CMD echo MYPATH
CMD echo "end..."
CMD /bin/bash
```

```shell
docker build -f /home/dockerfile-test/dockerfile -t centos:0.1 .
```

## 构建 Tomcat 镜像

### 1、前提准备

```shell
[ryo@localhost tomcat]$ ll
总用量 199844
-rw-rw-r--. 1 ryo ryo  10595340 8月  22 13:38 apache-tomcat-8.5.81.tar.gz
-rw-rw-r--. 1 ryo ryo 194042837 8月  22 13:38 jdk-8u202-linux-x64.tar.gz
-rw-rw-r--. 1 ryo ryo         0 8月  22 13:39 readme.txt
```

### 2、编写 DockerFile 文件

```

```
