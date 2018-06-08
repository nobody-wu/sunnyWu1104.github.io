---
title: Docker入门命令
tags: Docker
categories: Docker
copyright : ture
---

![Docker](http://p95stksgt.bkt.clouddn.com/docker01.png)

---

# docker

- 查看当前系统Docker信息:

```
docker info

```

---

# image

- 查找Docker Hub上的nginx镜像

```
docker search [image_name]

```

- 拉取docker镜像

```
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]

options:
- -a stdin: 指定标准输入输出内容类型，可选 STDIN/STDOUT/STDERR 三项；
- -d: 后台运行容器，并返回容器 ID；
- -i: 以交互模式运行容器，通常与 -t 同时使用；
- -t: 为容器重新分配一个伪输入终端，通常与 -i 同时使用；
- --name="nginx-lb": 为容器指定一个名称；
- --dns 8.8.8.8: 指定容器使用的 DNS 服务器，默认和宿主一致；
- --dns-search example.com: 指定容器 DNS 搜索域名，默认和宿主一致；
- -h "mars": 指定容器的 hostname；
- -e username="ritchie": 设置环境变量；
- --env-file=[]: 从指定文件读入环境变量；
- --cpuset="0-2" or --cpuset="0,1,2": 绑定容器到指定 CPU 运行；
- -m : 设置容器使用内存最大值；
- --net="bridge": 指定容器的网络连接类型，支持 bridge/host/none/container: 四种类型；
- --link=[]: 添加链接到另一个容器；
- --expose=[]: 开放一个端口或一组端口；


```

- 启动镜像

```
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]

```


- 查看宿主机上的镜像，Docker镜像保存在/var/lib/docker目录下:

```
docker images

或者

docker image list

或者

docker image ls

```

- 创建镜像文件

```
$ docker image build -t [image_name]

或者

$ docker image build -t [image_name]:[tag]

```

- 删除镜像

```
docker rmi  docker.io/tomcat:7.0.77-jre7   或者  docker rmi [image_id]

```

---


# container

- 生成容器

```
$ docker container run -p [本机端口号]:[容器端口号] -it [image_name] /bin/bash

或者

$ docker container run -p 8[本机端口号]:[容器端口号] -it [image_name]:[tag] /bin/bash

```

```
- -p参数：容器的 3000 端口映射到本机的 8000 端口。
- -it参数：容器的 Shell 映射到当前的 Shell，然后你在本机窗口输入的命令，就会传入容器。
- /bin/bash：容器启动以后，内部第一个执行的命令。这里是启动 Bash，保证用户可以使用 Shell。

```

- 查看当前有哪些容器正在运行

```
docker ps

```

- 查看所有容器

```
docker ps -a

```

- 终止容器运行:

```
docker container kill [containerID]

```


- 启动、停止、重启容器命令：

```
docker run  --name [自己定义的容器名称]  -d -p [本机端口号]:[容器端口号] [image_name]:[tag]

docker start [container_name]/[container_id]
docker stop [container_name]/[container_id]
docker restart [container_name]/[container_id]

```

- 后台启动一个容器后，如果想进入到这个容器，可以使用attach命令：

```
docker attach [container_name]/[container_id]

或者

docker container exec -it [containerID] /bin/bash （进入一个正在运行的 docker 容器。如果docker run命令运行容器的时候，没有使用-it参数，就要用这个命令进入容器。一旦进入了容器，就可以在容器的 Shell 执行命令了。）

或者

docker exec -it [自己命名的容器名称] bash

```

- 删除容器的命令：

```
docker rm [container_name]/[container_id]

```

- 删除所有停止的容器：

```
docker rm $(docker ps -a -q)

```

- 在容器终止运行后自动删除容器文件:

```
docker container run --rm -p 8000:3000 -it koa-demo /bin/bash

```

- 从正在运行的 Docker 容器里面，将文件拷贝到本机：

```
docker container cp [containID]:[/path/to/file]

```

---

# push image

```
1、 docker login

2、docker image tag [imageName] [username]/[repository]:[tag]
   docker image tag koa-demos:0.0.1 ruanyf/koa-demos:0.0.1

3、 docker image build -t [username]/[repository]:[tag]

4、docker image push [username]/[repository]:[tag]

```