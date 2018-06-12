---
title: Docker三剑客之Componse项目
tags: Docker
categories: Docker
copyright : ture
---

![Docker](http://p95stksgt.bkt.clouddn.com/docker01.png)

---

Docker Compose 是 Docker 官方编排（Orchestration）项目之一，负责快速的部署分布式应用。
本章将介绍 Compose 项目情况以及安装和使用。

Dockerfile 可以让用户管理一个单独的应用容器；而 Compose 则允许用户在一个模板（YAML 格式）中定义一组相关联的应用容器（被称为一个 project，即项目），例如一个 Web 服务容器再加上后端的数据库服务容器等。

通过 Docker-Compose 用户可以很容易地用一个配置文件定义一个多容器的应用，然后使用一条指令安装这个应用的所有依赖，完成构建。Docker-Compose 解决了容器与容器之间如何管理编排的问题。

Compose 中有两个重要的概念：
- 服务 (service) ：一个应用的容器，实际上可以包括若干运行相同镜像的容器实例。
- 项目 (project) ：由一组关联的应用容器组成的一个完整业务单元，在 docker-compose.yml 文件中定义。
一个项目可以由多个服务（容器）关联而成，Compose 面向项目进行管理，通过子命令对项目中的一组容器进行便捷地生命周期管理。
Compose 项目由 Python 编写，实现上调用了 Docker 服务提供的 API 来对容器进行管理。因此，只要所操作的平台支持 Docker API，就可以在其上利用 Compose 来进行编排管理。

> Docker一开始吸引我的点正是：Docker Componse(至少在没看到machine之前我是这么想的=。=、)，我觉得它完美的解决了服务器上一些服务的复杂部署。尤其是mysql、tomcat、nginx等等。Docker Componse就像哆啦A梦的神奇口袋（当然，官方镜像有时候并不能满足我们的需求，那就自己搞呗手动滑稽）

一张图了解一下原理：
<img src="http://p95stksgt.bkt.clouddn.com/docker-componse01.jpg" width="50%" height="50%">

# Docker Componse安装

```
方法一：
下载
sudo curl -L https://github.com/docker/compose/releases/download/1.20.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
安装
chmod +x /usr/local/bin/docker-compose

方法二：
#安装pip
yum -y install epel-release
yum -y install python-pip
确认版本
pip --version
更新pip
pip install --upgrade pip
安装docker-compose
pip install docker-compose

```

# 安装补全工具(可选)

为了方便我们输入命令，也可以安装 Docker 的补全提示工具帮忙我们快速输入命令

```
#安装
yum install bash-completion

#下载docker-compose脚本
curl -L https://raw.githubusercontent.com/docker/compose/$(docker-compose version --short)/contrib/completion/bash/docker-compose > /etc/bash_completion.d/docker-compose

```

# Docker Compose 常用命令

命令的基本的使用格式是:

```
docker-compose [-f=<arg>...] [options] [COMMAND] [ARGS...]

	- -f, --file FILE 指定使用的 Compose 模板文件，默认为 docker-compose.yml，可以多次指定。
	- -p, --project-name NAME 指定项目名称，默认将使用所在目录名称作为项目名。
	- --x-networking 使用 Docker 的可拔插网络后端特性
	- --x-network-driver DRIVER 指定网络后端的驱动，默认为 bridge
	- --verbose 输出更多调试信息。
	- -v, --version 打印版本并退出。

```

## build

构建（重新构建）项目中的服务容器

格式为 docker-compose build [options] [SERVICE...]

选项包括：
	- --force-rm 删除构建过程中的临时容器。
	- --no-cache 构建镜像过程中不使用 cache（这将加长构建过程）。
	- --pull 始终尝试通过 pull 来获取更新版本的镜像。
## config

验证 Compose 文件格式是否正确，若正确则显示配置，若格式错误显示错误原因。

格式为 docker-compose config

## down

此命令将会停止 up 命令所启动的容器，并移除网络

格式为 docker-compose down

## exec

进入指定的容器。

格式为 docker-compose exec

## images

列出 Compose 文件中包含的镜像。

格式为 docker-compose images

## kill

通过发送 SIGKILL 信号来强制停止服务容器。支持通过 -s 参数来指定发送的信号，例如通过如下指令发送 SIGINT 信号。

格式为 docker-compose kill [options] [SERVICE...]

## logs

查看服务容器的输出。默认情况下，docker-compose 将对不同的服务输出使用不同的颜色来区分。可以通过 --no-color 来关闭颜色。
该命令在调试问题的时候十分有用。

格式为 docker-compose logs [options] [SERVICE...]

## pause

暂停一个服务容器

格式为 docker-compose pause

## port

打印某个容器端口所映射的公共端口

格式为 docker-compose port [options] SERVICE PRIVATE_PORT

选项：
	- --protocol=proto 指定端口协议，tcp（默认值）或者 udp。
	- --index=index 如果同一服务存在多个容器，指定命令对象容器的序号（默认为 1）。
## ps

列出项目中目前的所有容器

格式为 docker-compose ps

选项：
	- -q 只打印容器的 ID 信息。

## pull

拉取服务依赖的镜像

格式为 docker-compose pull [options] [SERVICE...]

选项：
	- --ignore-pull-failures 忽略拉取镜像过程中的错误。

## push

推送服务依赖的镜像到 Docker 镜像仓库

格式为 docker-compose push


## restart

重启项目中的服务

格式为 docker-compose restart [options] [SERVICE...]

选项：
	- -t, --timeout TIMEOUT 指定重启前停止容器的超时（默认为 10 秒）。

## rm

删除所有（停止状态的）服务容器。推荐先执行 docker-compose stop 命令来停止容器

格式为 docker-compose rm [options] [SERVICE...]

选项：
	- -f, --force 强制直接删除，包括非停止状态的容器。一般尽量不要使用该选项。
	- -v 删除容器所挂载的数据卷。
## run

在指定服务上执行一个命令

格式为 docker-compose run [options] [-p PORT...] [-e KEY=VAL...] SERVICE [COMMAND] [ARGS...]

### eg:

```
docker-compose run ubuntu ping docker.com
将会启动一个 ubuntu 服务容器，并执行 ping docker.com 命令

```

默认情况下，如果存在关联，则所有关联的服务将会自动被启动，除非这些服务已经在运行中。
该命令类似启动容器后运行指定的命令，相关卷、链接等等都将会按照配置自动创建。

两个不同点：
	- 给定命令将会覆盖原有的自动运行命令；
	- 不会自动创建端口，以避免冲突。

如果不希望自动启动关联的容器，可以使用 --no-deps 选项，将不会启动 web 容器所关联的其它容器。

选项：
	- -d 后台运行容器。
	- --name NAME 为容器指定一个名字。
	- --entrypoint CMD 覆盖默认的容器启动指令。
	- -e KEY=VAL 设置环境变量值，可多次使用选项来设置多个环境变量。
	- -u, --user="" 指定运行容器的用户名或者 uid。
	- --no-deps 不自动启动关联的服务容器。
	- --rm 运行命令后自动删除容器，d 模式下将忽略。
	- -p, --publish=[] 映射容器端口到本地主机。
	- --service-ports 配置服务端口并映射到本地主机。
	- -T 不分配伪 tty，意味着依赖 tty 的指令将无法运行。

## scale

设置指定服务运行的容器个数，通过 service=num 的参数来设置数量。

格式为 docker-compose scale [options] [SERVICE=NUM...]

```
docker-compose scale web=3 db=2
将启动 3 个容器运行 web 服务，2 个容器运行 db 服务。

一般的，当指定数目多于该服务当前实际运行容器，将新创建并启动容器；反之，将停止容器。

```

选项：
	- -t, --timeout TIMEOUT 停止容器时候的超时（默认为 10 秒）。


## start

启动已经存在的服务容器

格式为 docker-compose start [SERVICE...]

## stop

停止已经处于运行状态的容器，但不删除它。通过 docker-compose start 可以再次启动这些容器

格式为 docker-compose stop [options] [SERVICE...]

选项：
	- -t, --timeout TIMEOUT 停止容器时候的超时（默认为 10 秒）。


## top

查看各个服务容器内运行的进程。

## unpause

恢复处于暂停状态中的服务

格式为 docker-compose unpause [SERVICE...]

## up

该命令十分强大，它将尝试自动完成包括构建镜像，（重新）创建服务，启动服务，并关联服务相关容器的一系列操作。

格式为 docker-compose up [options] [SERVICE...]

默认情况，如果服务容器已经存在，docker-compose up 将会尝试停止容器，然后重新创建（保持使用 volumes-from 挂载的卷），以保证新启动的服务匹配 docker-compose.yml 文件的最新内容。如果用户不希望容器被停止并重新创建，可以使用 docker-compose up --no-recreate。这样将只会启动处于停止状态的容器，而忽略已经运行的服务。如果用户只想重新部署某个服务，可以使用 docker-compose up --no-deps -d <SERVICE_NAME> 来重新创建服务并后台停止旧服务，启动新服务，并不会影响到其所依赖的服务。

选项：
	- -d 在后台运行服务容器。
	- --no-color 不使用颜色来区分不同的服务的控制台输出。
	- --no-deps 不启动服务所链接的容器。
	- --force-recreate 强制重新创建容器，不能与 --no-recreate 同时使用。
	- --no-recreate 如果容器已经存在了，则不重新创建，不能与 --force-recreate 同时使用。
	- --no-build 不自动构建缺失的服务镜像。
	- -t, --timeout TIMEOUT 停止容器时候的超时（默认为 10 秒）。

