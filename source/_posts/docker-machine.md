---
title: Docker三剑客之Machine项目
tags: Docker
categories: Docker
copyright : ture
---

![Docker](http://p95stksgt.bkt.clouddn.com/docker02.png)

---

> Docker Componse吸引我学习Docker，Machine令我爱上Docker，一定不要错过这两个伟大的项目。严肃脸:)


## Docker Machine 介绍

Docker Machine 是 Docker 官方编排（Orchestration）项目之一，负责在多种平台上快速安装 Docker 环境。

Docker Machine 是一个工具，它允许你在虚拟宿主机上安装 Docker Engine ，并使用 docker-machine 命令管理这些宿主机。你可以使用 Machine 在你本地的 Mac 或 Windows box、公司网络、数据中心、或像 AWS 或 Digital Ocean 这样的云提供商上创建 Docker 宿主机。（跨平台，跨环境）

使用 docker-machine 命令，你可以启动、审查、停止和重新启动托管的宿主机、升级 Docker 客户端和守护程序、并配置 Docker 客户端与你的宿主机通信。（集中管理）

> Docker Machine 项目基于 Go 语言实现（习惯面向对象的同学，学习go的基础语法，不超过一周），目前在 [Github](https://github.com/docker/machine) 上进行维护。

### 为什么要使用它？

1. 跨平台跨环境运行Docker

Machine 允许你在较早的 Mac 或 Windows 系统上运行 Docker

![docker-machine01](http://p95stksgt.bkt.clouddn.com/docker-machine01.jpg)

如果你主要在不符合新的 Docker for Mac 和 Docker for Windows 应用程序的旧 Mac 或 Windows 笔记本电脑或台式机上工作，则需要 Docker Machine 来在本地“运行Docker”（即Docker Engine）。在 Mac 或 Windows box 中使用 Docker Toolbox 安装程序安装 Docker Machine 将为 Docker Engine 配置一个本地的虚拟机，使你能够连接它、并运行 docker 命令。

2. 远程系统集中配置Docker宿主机

![docker-machine02](http://p95stksgt.bkt.clouddn.com/docker-machine02.jpg)


Docker Engine Linux 系统上原生地运行。如果你有一个 Linux 作为你的主系统，并且想要运行 docker 命令，所有你需要做的就是下载并安装 Docker Engine 。然而，如果你想要在网络上、云中甚至本地配置多个 Docker 宿主机有一个有效的方式，你需要 Docker Machine。

无论你的主系统是 Mac、Windows 还是 Linux，你都可以在其上安装 Docker Machine，并使用 docker-machine 命令来配置和管理大量的 Docker 宿主机。它会自动创建宿主机、在其上安装 Docker Engine 、然后配置 docker 客户端。每个被管理的宿主机（“machine”）是 Docker 宿主机和配置好的客户端的结合。


### Docker Engine 和 Docker Machine 有什么区别？

首先，Docker Engine与Docker Machine都有自己的命令行客户端。但是Docker Machine 是一个用于配置和管理你的宿主机（上面具有 Docker Engine 的主机）的工具，可以使用Docker Machine在一个或多个虚拟系统上安装Docker Engine。

这些虚拟系统可以是本地的（就像你在 Mac 或 Windows 上使用 Machine 在 VirtualBox 中安装和运行 Docker Engine 一样）或远程的（就像你使用 Machine 在云提供商上 provision Dockerized 宿主机一样）。Dockerized 宿主机本身可以认为是，且有时就称为，被管理的“machines”。

eg:
- 本地系统安装Docker Machine
- 利用Docker Machine创建虚拟机
- 在虚拟机上就可以玩耍运行 Docker Engine

## 安装Docker Machine

[docker-machine-release](https://github.com/docker/machine/releases)

On Linux:
```
$ curl -L https://github.com/docker/machine/releases/download/v0.14.0/docker-machine-`uname -s`-`uname -m` >/tmp/docker-machine &&
    chmod +x /tmp/docker-machine &&
    sudo cp /tmp/docker-machine /usr/local/bin/docker-machine
```
On OS X:
```
$ curl -L https://github.com/docker/machine/releases/download/v0.14.0/docker-machine-`uname -s`-`uname -m` >/usr/local/bin/docker-machine && \
  chmod +x /usr/local/bin/docker-machine
```

On Windows with git bash:
```
$ if [[ ! -d "$HOME/bin" ]]; then mkdir -p "$HOME/bin"; fi && \
curl -L https://github.com/docker/machine/releases/download/v0.14.0/docker-machine-Windows-x86_64.exe > "$HOME/bin/docker-machine.exe" && \
chmod +x "$HOME/bin/docker-machine.exe"

```

## 使用Docker Machine

Docker Machine 支持多种后端驱动，包括虚拟机、本地主机和云平台等。

### 创建本地主机实例:Virtualbox 驱动

#### Linux 17.10安装VirtualBox
这里演示的是一个通过Docker Machine创建一个VirtualBox，但是想要创建VirtualBox，首先需要当前主机已经安装了virtualbox才可以继续。

如果你使用Mac OS/Windows，那么很幸运，你只需要去VirtualBox的官网下载对应的包即可。如果使用的是Linux操作系统，那么会很麻烦。我这边展示一下使用Linux如果安装VirtualBox（毕竟，方便的事做了也没意思:)）

我的操作系统版本：Linux 17.10

1.  Setup Apt Repository

打开/etc/apt/source.list文件，添加以下代码:

```
For Ubuntu 17.10 ("artful")
deb http://download.virtualbox.org/virtualbox/debian artful contrib

For Ubuntu 17.04 ("Zesty")
deb http://download.virtualbox.org/virtualbox/debian zesty contrib

For Ubuntu 16.04 ("Xenial")
deb http://download.virtualbox.org/virtualbox/debian xenial contrib

For Ubuntu 14.04 ("Trusty")
deb http://download.virtualbox.org/virtualbox/debian trusty contrib

For Ubuntu 12.04 LTS ("Precise Pangolin")
deb http://download.virtualbox.org/virtualbox/debian precise contrib

For Debian 8 ("Jessie")
deb http://download.virtualbox.org/virtualbox/debian jessie contrib

For Debian 7 ("Wheezy")
deb http://download.virtualbox.org/virtualbox/debian wheezy contrib


```

不知道名称的，出门左转[virtualbox](http://download.virtualbox.org/virtualbox/debian/dists/)

2. Setup Oracle public key

添加公钥使其信任

```
$ curl -fsS https://www.virtualbox.org/download/oracle_vbox_2016.asc -O- | sudo apt-key add -
$ curl -fsS https://www.virtualbox.org/download/oracle_vbox.asc -O- | sudo apt-key add -

```

这一步和安装Docker时添加官方GPG密钥一致

3. Install Oracle VirtualBox

愉快的安装

```
sudo apt-get update
sudo apt-get install virtualbox-5.1

```

#### 创建一个虚拟机

安装完VirtualBox后，继续创建虚拟主机

使用 virtualbox 类型的驱动，创建一台 Docker 主机，命名为 default。

```
docker-machine create --driver virtualbox default


```

也可以在创建时加上如下参数，来配置主机或者主机上的 Docker:

- --engine-opt dns=114.114.114.114 配置 Docker 的默认 DNS
- --engine-registry-mirror https://registry.docker-cn.com 配置 Docker 的仓库镜像
- --virtualbox-memory 2048 配置主机内存
- --virtualbox-cpu-count 2 配置主机 CPU

ps:

前一天刚实验完，可以创建在虚拟机内安装并虚拟机，但是今天却再次创建虚拟机的时候就报错:

```
This computer doesn't have VT-X/AMD-v enabled. Enabling it in the BIOS is mandatory

```

这表示系统没有开启Intel Virtualization Technology，可是我偏偏该死的用的Virtualbox，这个虚拟机只有EFI(虽然类似bios，但是少了很多选项)并没有bios，试着半天没有办法继续下去。（命令行直接修改没有试过行不行，应该是不行的）

后来google告诉我虚拟机内是不支持虚拟机的虚拟化。。。要这么说的话，云服务器&云主机&虚拟主机通过虚拟化手段的主机都不能用了。难道我昨天活在梦里了？？？

这里就不再继续演示virtualbox的驱动了，大家可以找别的玩玩

	•	amazonec2
	•	azure
	•	digitalocean
	•	exoscale
	•	generic
	•	google
	•	hyperv
	•	none
	•	openstack
	•	rackspace
	•	softlayer
	•	virtualbox
	•	vmwarevcloudair
	•	vmwarefusion
	•	vmwarevsphere


### 创建本地主机实例:macOS xhyve 驱动

[xhyve 驱动](https://github.com/zchee/docker-machine-driver-xhyve)

xhyve 是 macOS 上轻量化的虚拟引擎，使用其创建的 Docker Machine 较 VirtualBox 驱动创建的运行效率要高。

表示没用过，做下记录。有需要的同学可以尝试一下。

```
$ brew install docker-machine-driver-xhyve

$ docker-machine create \
      -d xhyve \
      # --xhyve-boot2docker-url ~/.docker/machine/cache/boot2docker.iso \
      --engine-opt dns=114.114.114.114 \
      --engine-registry-mirror https://registry.docker-cn.com \
      --xhyve-memory-size 2048 \
      --xhyve-rawdisk \
      --xhyve-cpu-count 2 \
      xhyve

```

> 注意：非首次创建时建议加上 --xhyve-boot2docker-url ~/.docker/machine/cache/boot2docker.iso 参数，避免每次创建时都从 GitHub 下载 ISO 镜像。


### 创建本地主机实例:Windows 10

Windows 10 安装 Docker for Windows 之后不能再安装 VirtualBox，也就不能使用 virtualbox 驱动来创建 Docker Machine，我们可以选择使用 hyperv 驱动。

> 注意，必须事先在 Hyper-V 管理器中新建一个 外部虚拟交换机 执行下面的命令时，使用 --hyperv-virtual-switch=MY_SWITCH 指定虚拟交换机名称

```
docker-machine create --driver hyperv --hyperv-virtual-switch=MY_SWITCH vm

```

### 创建一个主机

```
docker-machine create --driver virtualbox default

```

这个命令会下载 boot2docker，基于 boot2docker 创建一个虚拟主机。boot2docker 是一个轻量级的 linux 发行版，基于专门为运行 docker 容器而设计的 Tiny Core Linux 系统，完全从 RAM 运行，45Mb左右，启动时间约5s。

服务列表:

```
docker-machine ls

```

创建主机成功后，可以通过 env 命令来让后续操作对象都是目标主机。

```
docker-machine env [主机名]
```

可以通过 SSH 登录到主机:

```
docker-machine ssh [主机名]
```

连接到主机之后你就可以在其上使用 Docker 了，退出虚拟机使用命令：exit 

## Docker Machine 常用命令

//创建虚拟机
docker-machine create [OPTIONS] [arg...]

//移除虚拟机
docker-machine rm [OPTIONS] [arg...]

//登录虚拟机
docker-machine ssh [arg...]

//docker客户端配置环境变量
docker-machine env [OPTIONS] [arg...]

//检查机子信息
docker-machine inspect

//查看虚拟机列表
docker-machine ls [OPTIONS] [arg...]

//查看虚拟机状态
docker-machine status [arg...]  //一个虚拟机名称

//启动虚拟机
docker-machine start [arg...]  //一个或多个虚拟机名称

//停止虚拟机
docker-machine stop [arg...]  //一个或多个虚拟机名称

//重启虚拟机
docker-machine restart [arg...]  //一个或多个虚拟机名称

## 更多命令

	•	active 查看活跃的 Docker 主机
	•	config 输出连接的配置信息
	•	create 创建一个 Docker 主机
	•	env 显示连接到某个主机需要的环境变量
	•	inspect 输出主机更多信息
	•	ip 获取主机地址
	•	kill 停止某个主机
	•	ls 列出所有管理的主机
	•	provision 重新设置一个已存在的主机
	•	regenerate-certs 为某个主机重新生成 TLS 认证信息
	•	restart 重启主机
	•	rm 删除某台主机
	•	ssh SSH 到主机上执行命令
	•	scp 在主机之间复制文件
	•	mount 挂载主机目录到本地
	•	start 启动一个主机
	•	status 查看主机状态
	•	stop 停止一个主机
	•	upgrade 更新主机 Docker 版本为最新
	•	url 获取主机的 URL
	•	version 输出 docker-machine 版本信息
	•	help 输出帮助信息

每个命令，又带有不同的参数，可以通过

```
$ docker-machine COMMAND --help
```

来查看具体的用法。


