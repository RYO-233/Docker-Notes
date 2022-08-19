[toc]

# DockerFile

> ​	DockerFile 是用来构建 Docker 镜像文件的。

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