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

To retrieve the join command including the join token for worker nodes, run the following command on a manager node:

```shell
$ docker swarm join-token worker

To add a worker to this swarm, run the following command:

    docker swarm join \
    --token SWMTKN-1-49nj1cmql0jkz5s954yi3oex3nedyz0fb0xx14ie39trti4wxv-8vxv8rssmk743ojnwacrr2e7c \
    192.168.99.100:2377
```

Run the command from the output on the worker to join the swarm:

```shell
$ docker swarm join \
  --token SWMTKN-1-49nj1cmql0jkz5s954yi3oex3nedyz0fb0xx14ie39trti4wxv-8vxv8rssmk743ojnwacrr2e7c \
  192.168.99.100:2377

This node joined a swarm as a worker.
```

The `docker swarm join` command does the following:

- switches the Docker Engine on the current node into swarm mode.
- requests a TLS certificate from the manager.
- names the node with the machine hostname
- joins the current node to the swarm at the manager listen address based upon the swarm token.
- sets the current node to `Active` availability, meaning it can receive tasks from the scheduler.
- extends the `ingress` overlay network to the current node.

#### 作为 manager node 加入

When you run `docker swarm join` and pass the manager token, the Docker Engine switches into swarm mode the same as for workers. Manager nodes also participate in the raft consensus. The new nodes should be `Reachable`, but the existing manager remains the swarm `Leader`.

Docker recommends three or five manager nodes per cluster to implement high availability. Because swarm mode manager nodes share data using Raft, there must be an odd number of managers. The swarm can continue to function after as long as a quorum of more than half of the manager nodes are available.

For more detail about swarm managers and administering a swarm, see [Administer and maintain a swarm of Docker Engines](https://docs.docker.com/engine/swarm/admin_guide/).

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

