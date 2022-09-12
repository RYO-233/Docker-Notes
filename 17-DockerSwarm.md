[toc]

# DockerSwarm

![swarm-diagram](img/swarm-diagram.png)

## Swarm 概念

### Swarm

> ​	使用[Swarmkit](https://github.com/moby/swarmkit)可以构建的集群管理和编排功能。Swarmkit 是一个单独的项目，它实现了 Docker 的编排层并直接在 Docker 中使用。
>
> ​	一个 swarm 由多个 Docker 主机组成，它们以 **swarm 模式** 运行并充当管理器（管理成员资格和委托）和工作人员（运行 [swarm 服务](https://docs.docker.com/engine/swarm/key-concepts/#services-and-tasks)）。给定的 Docker 主机可以是管理员、工作人员或同时执行这两种角色。

### Node

> ​	**节点**是参与 swarm 的 Docker 引擎的一个实例。

### Service

> ​	**服务**是要在管理节点或工作节点上执行的任务的定义。它是 swarm 系统的中心结构，也是用户与 swarm 交互的主要根源。
>
> ​	创建服务时，您需要指定要使用的容器映像以及在运行的容器中执行的命令。
>
> ​	在**复制服务**模型中，群管理器根据您在所需状态中设置的规模在节点之间分配特定数量的副本任务。
>
> ​	对于**全局服务**，swarm 在集群中的每个可用节点上为服务运行一个任务。

### Task

> ​	一个**任务**携带一个 Docker 容器和在容器内运行的命令。它是 swarm 的原子调度单元。Manager 节点根据服务规模中设置的副本数将任务分配给工作节点。一旦任务被分配给一个节点，它就不能移动到另一个节点。它只能在分配的节点上运行或失败。

## Swarm 搭建

### 创建 swarm

该`--advertise-addr`标志将管理节点配置为将其地址发布为`192.168.99.100`. swarm 中的其他节点必须能够访问该 IP 地址的管理器。

输出包括将新节点加入 swarm 的命令。根据`--token` 标志的值，节点将作为管理者或工作者加入。

```shell
docker swarm init --advertise-addr 192.168.200.151
```

### 查看 swarm 当前状态

```shell
 docker info
```

### 查看 nodes 信息

```shell
 docker node ls
```

### 添加 nodes 到 swarm

#### 作为 worker node 加入

> ​	要检索 worker 节点的 join 命令，包括 join 令牌，在管理节点上运行以下命令:

```shell
$ docker swarm join-token worker

To add a worker to this swarm, run the following command:

    docker swarm join \
    --token SWMTKN-1-49nj1cmql0jkz5s954yi3oex3nedyz0fb0xx14ie39trti4wxv-8vxv8rssmk743ojnwacrr2e7c \
    192.168.99.100:2377
```

> ​	在 worker 的回显信息中输入加入集群的命令:

```shell
$ docker swarm join \
  --token SWMTKN-1-49nj1cmql0jkz5s954yi3oex3nedyz0fb0xx14ie39trti4wxv-8vxv8rssmk743ojnwacrr2e7c \
  192.168.99.100:2377

This node joined a swarm as a worker.
```

' docker swarm join '命令执行如下操作:

- 将当前节点上的Docker引擎切换为集群模式。
- 向管理器请求TLS证书。
- 用机器主机名命名节点
- 根据群令牌在manager监听地址加入当前节点到群中。
- 设置当前节点为' Active '可用性，这意味着它可以从调度器接收任务。
- 扩展' ingress '覆盖网络到当前节点

#### 作为 manager node 加入

当你运行`docker swarm join`并传递经理令牌时，docker Engine 会切换到 swarm 模式，就像workers 一样。经理节点也参与筏子共识。新的节点应该是“可到达的”，但现有的管理器仍然是集群的“领导者”。

Docker 建议每个集群有 3 到 5 个管理节点来实现高可用性。因为群模式管理器节点使用 Raft 共享数据，所以必须有奇数个管理器。只要管理节点的仲裁数超过一半，集群就可以继续工作。

有关蜂群管理器和管理蜂群的更多细节，请参见[Administer and maintain a swarm of Docker Engines](https://docs.docker.com/engine/swarm/admin_guide/).

To retrieve the join command including the join token for manager nodes, run the following command on a manager node:

```
$ docker swarm join-token manager

To add a manager to this swarm, run the following command:

    docker swarm join \
    --token SWMTKN-1-61ztec5kyafptydic6jfc1i33t37flcl4nuipzcusor96k7kby-5vy9t8u35tuqm7vh67lrz9xp6 \
    192.168.99.100:2377
```

Run the command from the output on the new manager node to join it to the swarm:

```shell
$ docker swarm join \
  --token SWMTKN-1-61ztec5kyafptydic6jfc1i33t37flcl4nuipzcusor96k7kby-5vy9t8u35tuqm7vh67lrz9xp6 \
  192.168.99.100:2377
```

## Raft 一致性协议

[https://docs.docker.com/engine/swarm/raft/](https://docs.docker.com/engine/swarm/raft/)

## Swarm 弹性创建服务

[https://docs.docker.com/engine/swarm/services/](https://docs.docker.com/engine/swarm/services/)

## Docker Stack

[https://docs.docker.com/engine/swarm/stack-deploy/](https://docs.docker.com/engine/swarm/stack-deploy/)

## Docker Secret



## Docker Config

