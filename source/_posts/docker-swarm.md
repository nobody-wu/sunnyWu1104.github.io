---
title: Docker三剑客之Swarm项目
tags: Docker
categories: Docker
copyright: ture
---

> Docker Machine让我们更便捷的添加处理主机，Docker Swarm让我们更加方便的管理主机

## 为什么存在Swarm项目

Docker Swarm 是 Docker 官方三剑客项目之一，提供 Docker 容器集群服务，是 Docker 官方对容器云生态进行支持的核心方案。

使用它，用户可以将多个 Docker 主机封装为单个大型的虚拟 Docker 主机，快速打造一套容器云平台。

> 注意：Docker 1.12 Swarm mode 已经内嵌入 Docker 引擎，成为了 docker 子命令 docker swarm。请注意与旧的 Docker Swarm 区分开来。

Swarm mode 内置 kv 存储功能，提供了众多的新特性，比如：具有容错能力的去中心化设计、内置服务发现、负载均衡、路由网格、动态伸缩、滚动更新、安全传输等。使得 Docker 原生的 Swarm 集群具备与 Mesos、Kubernetes 竞争的实力。

## Docker Swarm的基本概念

Swarm 是使用 SwarmKit 构建的 Docker 引擎内置（原生）的集群管理和编排工具。

### 节点

运行 Docker 的主机可以主动初始化一个 Swarm 集群或者加入一个已存在的 Swarm 集群，这样这个运行 Docker 的主机就成为一个 Swarm 集群的节点 (node) 。

**节点分为管理 (manager) 节点和工作 (worker) 节点。**

管理节点用于 Swarm 集群的管理，docker swarm 命令基本只能在管理节点执行（节点退出集群命令 docker swarm leave 可以在工作节点执行）。一个 Swarm 集群可以有多个管理节点，但只有一个管理节点可以成为 leader，leader 通过 raft 协议实现。

工作节点是任务执行节点，管理节点将服务 (service) 下发至工作节点执行。管理节点默认也作为工作节点。你也可以通过配置让服务只运行在管理节点。

![docker-swarm01](http://p95stksgt.bkt.clouddn.com/docker-swarm01.jpg)

### 服务和任务

任务 （Task）是 Swarm 中的最小的调度单位，目前来说就是一个单一的容器。

服务 （Services） 是指一组任务的集合，服务定义了任务的属性。

服务有两种模式：

- replicated services 按照一定规则在各个工作节点上运行指定个数的任务。

- global services 每个工作节点上运行一个任务

两种模式通过 docker service create 的 --mode 参数指定。

![docker-swarm02](http://p95stksgt.bkt.clouddn.com/docker-swarm02.jpg)

## 创建Swarm集群

### 初始化集群

#### 使用 virtualbox 创建管理节点
```
docker-machine create --driver virtualbox manager1
#进入管理节点
docker-machine ssh manager1
```

> 执行 sudo -i 可以进入Root 权限

使用 docker swarm init 在 manager1 初始化一个 Swarm 集群

```
docker@manager1:~$ docker swarm init --advertise-addr 192.168.99.100
```

如果你的 Docker 主机有多个网卡，拥有多个 IP，必须使用 –advertise-addr 指定 IP。 执行 docker swarm init 命令的节点自动成为管理节点。

命令 docker info 可以查看 swarm 集群状态:

```
Containers: 0
Running: 0
Paused: 0
Stopped: 0
  ...snip...
Swarm: active
  NodeID: dxn1zf6l61qsb1josjja83ngz
  Is Manager: true
  Managers: 1
  Nodes: 1
  ...snip...
```

命令 docker node ls 可以查看集群节点信息

```
docker@manager1:~$ docker node ls

```

退出manager1虚拟主机

```
docker@manager1:~$ exit
```

#### 增加工作节点

上一步初始化了一个 Swarm 集群，拥有了一个管理节点，在 Docker Machine 一节中我们了解到 Docker Machine 可以在数秒内创建一个虚拟的 Docker 主机，下面我们使用它来创建两个 Docker 主机，并加入到集群中。

- 创建虚拟主机 worker1

```
docker-machine create -d virtualbox worker1
```

- 进入虚拟主机 worker1

```
docker-machine ssh worker1
```

- 加入 swarm 集群

```
docker@worker1:~$ docker swarm join \
    --token SWMTKN-1-47z6jld2o465z30dl7pie2kqe4oyug4fxdtbgkfjqgybsy4esl-8r55lxhxs7ozfil45gedd5b8a \
    192.168.99.100:2377

```

- 退出虚拟主机

```
docker@worker1:~$ exit

```

- 创建虚拟主机 worker2 步骤同上1～4

两个工作节点添加完成。

#### 查看集群

进入管理节点：

```
docker-machine ssh manager1

```

宿主机子上查看虚拟主机:

```
 docker-machine ls
```

在主节点上面执行 docker node ls 查询集群主机信息:

```
docker node ls
```

到此集群便创建成功了！


## 部署服务

使用 docker service 命令来管理 Swarm 集群中的服务，该命令只能在管理节点运行。

### 新建服务

进入集群管理节点：

```
docker-machine ssh manager1

```

使用 docker 中国镜像:

```
docker search alpine
docker pull registry.docker-cn.com/library/alpine

```

在上一节创建的 Swarm 集群中运行一个名为 helloworld 服务:

```
docker service create --replicas 1 --name helloworld alpine ping baidu.com

rwpw7eij4v6h6716jvqvpxbyv
overall progress: 1 out of 1 tasks
1/1: running   [==================================================>]
verify: Service converged
```

命令解释：

- docker service create 命令创建一个服务
- --name 服务名称命名为 helloworld
- --replicas 设置启动的示例数
- alpine 指的是使用的镜像名称，ping ityouknow.com指的是容器运行的bash

使用命令 docker service ps rwpw7eij4v6h6716jvqvpxbyv 可以查看服务进展

```
docker service ps rwpw7eij4v6h6716jvqvpxbyv

ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE                ERROR               PORTS
rgroe3s9qa53        helloworld.1        alpine:latest       worker1            Running             Running about a minute ago
```

使用 docker service ls 来查看当前 Swarm 集群运行的服务:

```
 docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
yzfmyggfky8c        helloworld          replicated          0/1                 alpine:latest
```

### 监控集群状态

登录管理节点 manager1:

```
docker-machine ssh manager1

```

运行 docker service inspect --pretty <SERVICE-ID> 查询服务概要状态，以 helloworld 服务为例：

```
运行 docker service inspect helloworld 查询服务详细信息
docker service inspect --pretty helloworld

ID:             rwpw7eij4v6h6716jvqvpxbyv
Name:           helloworld
Service Mode:   Replicated
 Replicas:      1
Placement:
UpdateConfig:
 Parallelism:   1
 On failure:    pause
 Monitoring Period: 5s
 Max failure ratio: 0
 ...
 Rollback order:    stop-first
ContainerSpec:
 Image:         alpine:latest@sha256:7b848083f93822dd21b0a2f14a110bd99f6efb4b838d499df6d04a49d0debf8b
 Args:          ping ityouknow.com
Resources:
Endpoint Mode:  vip
```

运行docker service ps <SERVICE-ID>  查看那个节点正在运行服务:

```
docker service ps helloworld

NAME                                    IMAGE   NODE     DESIRED STATE  LAST STATE
helloworld.1.8p1vev3fq5zm0mi8g0as41w35  alpine  worker1  Running        Running 3 minutes
```

在工作节点查看任务的执行情况:

```
docker-machine ssh  worker1

```

在节点执行docker ps 查看容器的运行状态:

```
docker ps

CONTAINER ID        IMAGE               COMMAND                CREATED             STATUS              PORTS               NAMES
96bf5b1d8010        alpine:latest       "ping ityouknow.com"   4 minutes ago       Up 4 minutes
```

在 Swarm 集群中成功的运行了一个 helloworld 服务，根据命令可以看出在 worker1 节点上运行。

## 弹性伸缩实验

可以使用 docker service scale 对一个服务运行的容器数量进行伸缩。

- 当业务处于高峰期时，我们需要扩展服务运行的容器数量:

```
docker service scale nginx=5

等同于

docker service update --replicas 5 helloworld

```

- 当业务平稳时，我们需要减少服务运行的容器数量:

```
docker service scale nginx=2

等同于

docker service update --replicas 2 helloworld

```

- 删除集群服务:

```
docker service rm helloworld

```

- 调整集群大小:

```
1. 创建虚拟主机 worker3
 docker-machine create -d virtualbox worker3

2. 进入虚拟主机 worker3
 docker-machine ssh worker3

3. 加入swarm 集群
 docker swarm join \
    --token SWMTKN-1-47z6jld2o465z30dl7pie2kqe4oyug4fxdtbgkfjqgybsy4esl-8r55lxhxs7ozfil45gedd5b8a \
    192.168.99.100:2377

```

这时进入manager主机查看node，看出集群节点多了 worker3。

- 退出 Swarm 集群:

进入对应的work主机后，执行:

```
docker swarm leave
```

- 重新搭建命令

使用 VirtualBox 做测试的时候，如果想重复实验可以将实验节点删掉再重来。

```
//停止虚拟机
docker-machine stop [arg...]  //一个或多个虚拟机名称

docker-machine stop   manager1 worker1 worker2

//移除虚拟机
docker-machine rm [OPTIONS] [arg...]

docker-machine rm manager1 worker1 worker2
```

停止、删除虚拟主机后，再重新创建即可。

## 在 Swarm 集群中使用 compose 文件

之前使用 docker-compose.yml 来一次配置、启动多个容器，在 Swarm 集群中也可以使用 compose 文件 （docker-compose.yml） 来配置、启动多个服务。

以在 Swarm 集群中部署 WordPress 为例进行说明:

```
version: "3"

services:
  wordpress:
    image: wordpress
    ports:
      - 80:80
    networks:
      - overlay
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress
    deploy:
      mode: replicated
      replicas: 3

  db:
    image: mysql
    networks:
       - overlay
    volumes:
      - db-data:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: somewordpress
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress
    deploy:
      placement:
        constraints: [node.role == manager]

  visualizer:
    image: dockersamples/visualizer:stable
    ports:
      - "8080:8080"
    stop_grace_period: 1m30s
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock"
    deploy:
      placement:
        constraints: [node.role == manager]

volumes:
  db-data:
networks:
  overlay:
```

### 部署compose

部署服务使用 docker stack deploy，其中 -c 参数指定 compose 文件名。

```
docker stack deploy -c docker-compose.yml wordpress
```

打开浏览器输入 任一节点IP:8080 即可看到各节点运行状态。如下图所示：

![docker-swarm03](http://p95stksgt.bkt.clouddn.com/docker-swarm03.png)

### 查看服务

```
docker stack ls

NAME                SERVICES
wordpress           3
```

### 移除服务

```
docker stack down wordpress
```

在 Swarm 集群管理节点新建该文件，其中的 visualizer 服务提供一个可视化页面，我们可以从浏览器中很直观的查看集群中各个服务的运行节点。

在 Swarm 集群中使用 docker-compose.yml 我们用 docker stack 命令，下面我们对该命令进行详细讲解。

## 在 Swarm 集群中管理敏感数据

在动态的、大规模的分布式集群上，管理和分发 密码、证书 等敏感信息是极其重要的工作。传统的密钥分发方式（如密钥放入镜像中，设置环境变量，volume 动态挂载等）都存在着潜在的巨大的安全风险。

Docker 目前已经提供了 secrets 管理功能，用户可以在 Swarm 集群中安全地管理密码、密钥证书等敏感数据，并允许在多个 Docker 容器实例之间共享访问指定的敏感数据。

> **注意： secret 也可以在 Docker Compose 中使用。**

### 创建 secret

- 创建文件 password.txt，里面存入 Mysql 的 root 密码
- 创建文件 wordpress.yml，用于启动 mysql 和 wordpress 服务，内容：

```
version: '3.3'

services:
   db:
     image: mysql:latest
     environment:
       MYSQL_ROOT_PASSWORD_FILE: /run/secrets/db_password
       MYSQL_DATABASE: wordpress
     secrets:
       - db_password

   wordpress:
     depends_on:
       - db
     image: wordpress:latest
     ports:
       - "8000:80"
     environment:
       WORDPRESS_DB_HOST: db:3306
       WORDPRESS_DB_USER: root
       WORDPRESS_DB_PASSWORD_FILE: /run/secrets/db_password
     secrets:
       - db_password

secrets:
   db_password:
     file: password.txt
```

启动 wordpress 服务:

```
docker stack deploy -c wordpress.yml wordpress
```

除了在Docker Componse里使用，也可以使用 docker secret create 命令以管道符的形式创建 secret

```
$ openssl rand -base64 20 | docker secret create mysql_password -

$ openssl rand -base64 20 | docker secret create mysql_root_password -
```

创建 MySQL 服务:

```
$ docker network create -d overlay mysql_private

$ docker service create \
     --name mysql \
     --replicas 1 \
     --network mysql_private \
     --mount type=volume,source=mydata,destination=/var/lib/mysql \
     --secret source=mysql_root_password,target=mysql_root_password \
     --secret source=mysql_password,target=mysql_password \
     -e MYSQL_ROOT_PASSWORD_FILE="/run/secrets/mysql_root_password" \
     -e MYSQL_PASSWORD_FILE="/run/secrets/mysql_password" \
     -e MYSQL_USER="wordpress" \
     -e MYSQL_DATABASE="wordpress" \
     mysql:latest
```

如果没有在 target 中显式的指定路径时，secret 默认通过 tmpfs 文件系统挂载到容器的 /run/secrets 目录中。

```
docker service create \
     --name wordpress \
     --replicas 1 \
     --network mysql_private \
     --publish target=30000,port=80 \
     --mount type=volume,source=wpdata,destination=/var/www/html \
     --secret source=mysql_password,target=wp_db_password,mode=0400 \
     -e WORDPRESS_DB_USER="wordpress" \
     -e WORDPRESS_DB_PASSWORD_FILE="/run/secrets/wp_db_password" \
     -e WORDPRESS_DB_HOST="mysql:3306" \
     -e WORDPRESS_DB_NAME="wordpress" \
     wordpress:latest
```

通过以上方法，没有像以前通过设置环境变量来设置 MySQL 密码， 而是采用 docker secret 来设置密码，防范了密码泄露的风险。

## 总结

在公司流量爆发的时候，只需要执行一个命令就可以完成实例上线。如果再根据公司的业务流量做自动化控制，那就真正实现了完全自动的动态伸缩。


