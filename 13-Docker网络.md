[toc]

# Docker网络

## Docker0

### ip addr

> ​	查询 IP 地址。
>
> ​	容器启动时，会得到一个 eth0@if25 的 IP 地址。

- lo：本地回环地址。

- ens33：阿里云内网地址。

- docker0：Docker 地址。

### Docker 是如何处理容器之间的网络访问的？

#### 原理

1. 每启动一个 Docker 容器，Docker 就会给 Docker 容器分配一个 IP，只要安装了 Docker，就会有一个网卡 docker0 桥接模式。使用的技术为 veth-pair 技术。
2. 容器所带的网卡，都是一对一对的。
   veth-pair 就是一对的虚拟设备接口，他们都是成对出现的，一段连着协议，一段连着接口。
   由于这个特性，veth-pair 充当一个桥梁，连接各种虚拟网络设备。

## --link

> ​	如果直接 ping 另一个容器，则会提示：
>
> ​		ping: \<docker container>: Name or service not known

```shell
docker run -d --name <container_name> -p 8086:8080 --link <link-container_name>:<another_name> <image>
```

### link 与 /etc/hosts

​	使用 link 机制，可以通过指定的名字来和目标容器通信，这其实是通过给`/etc/hosts`中加入名称和IP的解析关系来实现的。

## 自定义网络

### docker network

> ​	查看所有的 网络。
>
> ​	网络模式：
> ​		bridge：桥接模式。
> ​		none：不配置网络。
> ​		host：和宿主机共享网络。
> ​		container：容器网络连通。

```shell
docker network --help
```

| 参数       | 指令                                                 |
| ---------- | ---------------------------------------------------- |
| connect    | Connect a container to a network                     |
| create     | Create a network<br                                  |
| disconnect | Disconnect a container from a network                |
| inspect    | Display detailed information on one or more networks |
| ls         | List networks                                        |
| prune      | Remove all unused networks                           |
| rm         | Remove one or more networks                          |

### 使用自定义网络

```shell
docker run -it --network=my_net <container>

docker run -it --new my_net <container>
```

### 好处

> ​	自定义网络下的容器即可直接连接。

## 网络连通

| 参数    | 指令                             |
| ------- | -------------------------------- |
| connect | Connect a container to a network |

```
docker network connect mynetwork tomcat01
```

使用`docker network connect`，可以看到 centos01 直接被加入到了 mynetwork 这个网络里。这样 centos01 这个容器就拥有了两个 IP 地址，它在 docker0 中有一个，在 mynetwork 中还有一个。
