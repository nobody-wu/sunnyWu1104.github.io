---
title: Dockerfile命令详解
tags: Docker
categories: Docker
copyright : ture
---

![Docker](http://p95stksgt.bkt.clouddn.com/docker01.png)

---

## Dockerfile是什么？

简单说就是：镜像的定制

Docker 镜像是一个特殊的文件系统，除了提供容器运行时所需的程序、库、资源、配置等文件外，还包含了一些为运行时准备的一些配置参数（如匿名卷、环境变量、用户等）。镜像不包含任何动态数据，其内容在构建之后也不会被改变。
镜像的定制实际上就是定制每一层所添加的配置、文件。如果我们可以把每一层修改、安装、构建、操作的命令都写入一个脚本，用这个脚本来构建、定制镜像，那么之前提及的无法重复的问题、镜像构建透明性的问题、体积的问题就都会解决。这个脚本就是 Dockerfile。


## 为什么用Dockerfile？

Dockerfile其实是用于微服务化项目中镜像内容的处理方法。用来定义镜像、依赖镜像来运行容器。
使用Dockerfile非常容易的定义镜像内的内容。

Dockerfile 是一个文本文件，其内包含了一条条的指令(Instruction)，每一条指令构建一层，因此每一条指令的内容，就是描述该层应当如何构建。有了 Dockerfile，当我们需要定制自己额外的需求时，只需在 Dockerfile 上添加或者修改指令，重新生成 image 即可，省去了敲命令的麻烦。

换言之：我们可以通过一个简单的文件去创建镜像，启动容器等等的一系列脚本的动作。更方便的使我们部署项目

## Dcokerfile文件格式

```
##  Dockerfile文件格式

# This dockerfile uses the ubuntu image
# VERSION 2 - EDITION 1
# Author: docker_user
# Command format: Instruction [arguments / command] ..

# 1、第一行必须指定 基础镜像信息
FROM ubuntu

# 2、维护者信息
MAINTAINER docker_user docker_user@email.com

# 3、镜像操作指令
RUN echo "deb http://archive.ubuntu.com/ubuntu/ raring main universe" >> /etc/apt/sources.list
RUN apt-get update && apt-get install -y nginx
RUN echo "\ndaemon off;" >> /etc/nginx/nginx.conf

# 4、容器启动执行指令
CMD /usr/sbin/nginx

```

## Dcokerfile指令详解

###  FROM:指定基础镜像

FROM 指令用于指定其后构建新镜像所使用的基础镜像。FROM 指令必是 Dockerfile 文件中的首条命令，启动构建流程后，Docker 将会基于该镜像构建新镜像，FROM 后的命令也会基于这个基础镜像。

FROM语法格式为：

```
FROM <image>
或者
FROM <image>:<tag>
或者
FROM <image>:<digest>

```

通过 FROM 指定的镜像，可以是任何有效的基础镜像。FROM 有以下限制：
- 	FROM 必须 是 Dockerfile 中第一条非注释命令
- 在一个 Dockerfile 文件中创建多个镜像时，FROM 可以多次出现。只需在每个新命令 FROM 之前，记录提交(run)上次的镜像 ID。
- 	tag 或 digest 是可选的，如果不使用这两个值时，会使用 latest 版本的基础镜像

### RUN:执行命令

在镜像的构建过程中执行特定的命令，并生成一个中间镜像。格式:

```
#shell格式(主要还是用shell方便)
RUN <command>
#exec格式
RUN ["executable", "param1", "param2"]

```

- 	RUN 命令将在当前 image 中执行任意合法命令并提交执行结果。命令执行提交后，就会自动执行 Dockerfile 中的下一个指令。
- 层级 RUN 指令和生成提交是符合 Docker 核心理念的做法。它允许像版本控制那样，在任意一个点，对 image 镜像进行定制化构建。
- 	RUN 指令创建的中间镜像会被缓存，并会在下次构建中使用。如果不想使用这些缓存镜像，可以在构建时指定 --no-cache 参数，如：docker build --no-cache。

### COPY:复制文件

格式：

```
COPY <源路径>... <目标路径>
COPY ["<源路径1>",... "<目标路径>"]

使用通配符，其通配符规则要满足 Go 的 filepath.Match 规则
COPY hom* /mydir/
COPY hom?.txt /mydir/

eg:
COPY my.cnf /etc/my.cnf

```

### ADD:更高级的复制文件

ADD 指令和 COPY 的格式和性质基本一致。但是在 COPY 基础上增加了一些功能。比如<源路径>可以是一个 URL，这种情况下，Docker 引擎会试图去下载这个链接的文件放到<目标路径>去。

在构建镜像时，复制上下文中的文件到镜像内，格式：

```
ADD <源路径>... <目标路径>
ADD ["<源路径>",... "<目标路径>"]

```

* 如果 docker 发现文件内容被改变，则接下来的指令都不会再使用缓存。关于复制文件时需要处理的/，基本跟正常的 copy 一致

### ENV:设置环境变量

格式有两种：
```
ENV <key> <value>
ENV <key1>=<value1> <key2>=<value2>...

```

这个指令很简单，就是设置环境变量而已，无论是后面的其它指令，如 RUN，还是运行时的应用，都可以直接使用这里定义的环境变量。

```
ENV VERSION=1.0 DEBUG=on \
    NAME="Happy Feet"

```

这个例子中演示了如何换行，以及对含有空格的值用双引号括起来的办法，这和 Shell 下的行为是一致的。


### EXPOSE:设置监听端口

```
EXPOSE <port> [<port>...]

```

EXPOSE 指令并不会让容器监听 host 的端口，如果需要，需要在 docker run 时使用 -p、-P 参数来发布容器端口到 host 的某个端口上。

### VOLUME:定义匿名卷

VOLUME用于创建挂载点，即向基于所构建镜像创始的容器添加卷：

```
VOLUME ["/data"]

```

一个卷可以存在于一个或多个容器的指定目录，该目录可以绕过联合文件系统，并具有以下功能：

- 卷可以容器间共享和重用
- 容器并不一定要和其它容器共享卷
- 修改卷后会立即生效
- 对卷的修改不会对镜像产生影响
- 卷会一直存在，直到没有任何容器在使用它

* VOLUME 让我们可以将源代码、数据或其它内容添加到镜像中，而又不并提交到镜像中，并使我们可以多个容器间共享这些内容。

### WORKDIR:指定工作目录

WORKDIR用于在容器内设置一个工作目录：

```
WORKDIR /path/to/workdir

```

通过WORKDIR设置工作目录后，Dockerfile 中其后的命令 RUN、CMD、ENTRYPOINT、ADD、COPY 等命令都会在该目录下执行。 如，使用WORKDIR设置工作目录：

```
WORKDIR /a
WORKDIR b
WORKDIR c
RUN pwd

```

在以上示例中，pwd 最终将会在 /a/b/c 目录中执行。在使用 docker run 运行容器时，可以通过-w参数覆盖构建时所设置的工作目录。

### USER:指定当前用户

USER 用于指定运行镜像所使用的用户：

```
USER daemon

```

使用USER指定用户时，可以使用用户名、UID 或 GID，或是两者的组合。以下都是合法的指定试：

```
USER user
USER user:group
USER uid
USER uid:gid
USER user:gid
USER uid:group

```

使用USER指定用户后，Dockerfile 中其后的命令 RUN、CMD、ENTRYPOINT 都将使用该用户。镜像构建完成后，通过 docker run 运行容器时，可以通过 -u 参数来覆盖所指定的用户。

### CMD:指定在容器启动时所要执行的命令

有以下三种格式：

```
CMD ["executable","param1","param2"]
CMD ["param1","param2"]
CMD command param1 param2(shell)

```

省略可执行文件的 exec 格式，这种写法使 CMD 中的参数当做 ENTRYPOINT 的默认参数，此时 ENTRYPOINT 也应该是 exec 格式，具体与 ENTRYPOINT 的组合使用，参考 ENTRYPOINT。

* 注意
与 RUN 指令的区别：RUN 在构建的时候执行，并生成一个新的镜像，CMD 在容器运行的时候执行，在构建时不进行任何操作。

### ENTRYPOINT

ENTRYPOINT 用于给容器配置一个可执行程序。也就是说，每次使用镜像创建容器时，通过 ENTRYPOINT 指定的程序都会被设置为默认程序。ENTRYPOINT 有以下两种形式：

```
ENTRYPOINT ["executable", "param1", "param2"]
ENTRYPOINT command param1 param2

```

ENTRYPOINT 与 CMD 非常类似，不同的是通过docker run执行的命令不会覆盖 ENTRYPOINT，而docker run命令中指定的任何参数，都会被当做参数再次传递给 ENTRYPOINT。Dockerfile 中只允许有一个 ENTRYPOINT 命令，多指定时会覆盖前面的设置，而只执行最后的 ENTRYPOINT 指令。

docker run运行容器时指定的参数都会被传递给 ENTRYPOINT ，且会覆盖 CMD 命令指定的参数。如，执行docker run <image> -d时，-d 参数将被传递给入口点。

也可以通过docker run --entrypoint重写 ENTRYPOINT 入口点。如：可以像下面这样指定一个容器执行程序：

```
ENTRYPOINT ["/usr/bin/nginx"]

```

完整构建代码：

```
# Version: 0.0.3
FROM ubuntu:16.04
MAINTAINER 何民三 "cn.liuht@gmail.com"
RUN apt-get update
RUN apt-get install -y nginx
RUN echo 'Hello World, 我是个容器' \
   > /var/www/html/index.html
ENTRYPOINT ["/usr/sbin/nginx"]
EXPOSE 80

```

使用docker build构建镜像，并将镜像指定为 itbilu/test：

```
docker build -t="itbilu/test" .  ---后面的路径.别忘了

```

构建完成后，使用itbilu/test启动一个容器：

```
docker run -i -t  itbilu/test -g "daemon off;"

```

在运行容器时，我们使用了 -g "daemon off;"，这个参数将会被传递给 ENTRYPOINT，最终在容器中执行的命令为 /usr/sbin/nginx -g "daemon off;"


### LABEL

LABEL用于为镜像添加元数据，元数以键值对的形式指定：

```
LABEL <key>=<value> <key>=<value> <key>=<value> ...

```

使用LABEL指定元数据时，一条LABEL指定可以指定一或多条元数据，指定多条元数据时不同元数据之间通过空格分隔。推荐将所有的元数据通过一条LABEL指令指定，以免生成过多的中间镜像。 如，通过LABEL指定一些元数据：

```
LABEL version="1.0" description="这是一个Web服务器" by="IT笔录"

```

指定后可以通过docker inspect查看：

```
docker inspect itbilu/test
"Labels": {
    "version": "1.0",
    "description": "这是一个Web服务器",
    "by": "IT笔录"
},

```

### ARG

ARG用于指定传递给构建运行时的变量：

```
ARG <name>[=<default value>]

```

如，通过ARG指定两个变量：

```
ARG site
ARG build_user=IT笔录

```

以上我们指定了 site 和 build_user 两个变量，其中 build_user 指定了默认值。在使用 docker build 构建镜像时，可以通过 --build-arg <varname>=<value> 参数来指定或重设置这些变量的值。

```
docker build --build-arg site=itiblu.com -t itbilu/test .

```

这样我们构建了 itbilu/test 镜像，其中site会被设置为 itbilu.com，由于没有指定 build_user，其值将是默认值 IT 笔录。

### ONBUILD

ONBUILD用于设置镜像触发器：

```
ONBUILD [INSTRUCTION]

```

当所构建的镜像被用做其它镜像的基础镜像，该镜像中的触发器将会被钥触发。 如，当镜像被使用时，可能需要做一些处理：

```
[...]
ONBUILD ADD . /app/src
ONBUILD RUN /usr/local/bin/python-build --dir /app/src
[...]

```

* ps:因为在实际项目是依赖镜像的情况还是蛮多的，所以这个也会经常用到

### STOPSIGNAL

STOPSIGNAL用于设置停止容器所要发送的系统调用信号：

```
STOPSIGNAL signal

```

所使用的信号必须是内核系统调用表中的合法的值，如：SIGKILL。


### SHELL

SHELL用于设置执行命令（shell式）所使用的的默认 shell 类型：

```
SHELL ["executable", "parameters"]

```

SHELL在Windows环境下比较有用，Windows 下通常会有 cmd 和 powershell 两种 shell，可能还会有 sh。这时就可以通过 SHELL 来指定所使用的 shell 类型：

```
FROM microsoft/windowsservercore

# Executed as cmd /S /C echo default
RUN echo default

# Executed as cmd /S /C powershell -command Write-Host default
RUN powershell -command Write-Host default

# Executed as powershell -command Write-Host hello
SHELL ["powershell", "-command"]
RUN Write-Host hello

# Executed as cmd /S /C echo hello
SHELL ["cmd", "/S"", "/C"]
RUN echo hello

```

## Dockerfile示例

### 构建Nginx运行环境

```
# 指定基础镜像
FROM sameersbn/ubuntu:14.04.20161014

# 维护者信息
MAINTAINER sameer@damagehead.com

# 设置环境
ENV RTMP_VERSION=1.1.10 \
    NPS_VERSION=1.11.33.4 \
    LIBAV_VERSION=11.8 \
    NGINX_VERSION=1.10.1 \
    NGINX_USER=www-data \
    NGINX_SITECONF_DIR=/etc/nginx/sites-enabled \
    NGINX_LOG_DIR=/var/log/nginx \
    NGINX_TEMP_DIR=/var/lib/nginx \
    NGINX_SETUP_DIR=/var/cache/nginx

# 设置构建时变量，镜像建立完成后就失效
ARG BUILD_LIBAV=false
ARG WITH_DEBUG=false
ARG WITH_PAGESPEED=true
ARG WITH_RTMP=true

# 复制本地文件到容器目录中
COPY setup/ ${NGINX_SETUP_DIR}/
RUN bash ${NGINX_SETUP_DIR}/install.sh

# 复制本地配置文件到容器目录中
COPY nginx.conf /etc/nginx/nginx.conf
COPY entrypoint.sh /sbin/entrypoint.sh

# 运行指令
RUN chmod 755 /sbin/entrypoint.sh

# 允许指定的端口
EXPOSE 80/tcp 443/tcp 1935/tcp

# 指定网站目录挂载点
VOLUME ["${NGINX_SITECONF_DIR}"]

ENTRYPOINT ["/sbin/entrypoint.sh"]
CMD ["/usr/sbin/nginx"]

```

### 构建tomcat 环境

```
# 指定基于的基础镜像
FROM ubuntu:13.10

# 维护者信息
MAINTAINER zhangjiayang "zhangjiayang@sczq.com.cn"

# 镜像的指令操作
# 获取APT更新的资源列表
RUN echo "deb http://archive.ubuntu.com/ubuntu precise main universe"> /etc/apt/sources.list
# 更新软件
RUN apt-get update

# Install curl
RUN apt-get -y install curl

# Install JDK 7
RUN cd /tmp &&  curl -L 'http://download.oracle.com/otn-pub/java/jdk/7u65-b17/jdk-7u65-linux-x64.tar.gz' -H 'Cookie: oraclelicense=accept-securebackup-cookie; gpw_e24=Dockerfile' | tar -xz
RUN mkdir -p /usr/lib/jvm
RUN mv /tmp/jdk1.7.0_65/ /usr/lib/jvm/java-7-oracle/

# Set Oracle JDK 7 as default Java
RUN update-alternatives --install /usr/bin/java java /usr/lib/jvm/java-7-oracle/bin/java 300
RUN update-alternatives --install /usr/bin/javac javac /usr/lib/jvm/java-7-oracle/bin/javac 300

# 设置系统环境
ENV JAVA_HOME /usr/lib/jvm/java-7-oracle/

# Install tomcat7
RUN cd /tmp && curl -L 'http://archive.apache.org/dist/tomcat/tomcat-7/v7.0.8/bin/apache-tomcat-7.0.8.tar.gz' | tar -xz
RUN mv /tmp/apache-tomcat-7.0.8/ /opt/tomcat7/

ENV CATALINA_HOME /opt/tomcat7
ENV PATH $PATH:$CATALINA_HOME/bin

# 复件tomcat7.sh到容器中的目录
ADD tomcat7.sh /etc/init.d/tomcat7
RUN chmod 755 /etc/init.d/tomcat7

# Expose ports.  指定暴露的端口
EXPOSE 8080

# Define default command.
ENTRYPOINT service tomcat7 start && tail -f /opt/tomcat7/logs/catalina.out

```

---

tomcat7.sh命令文件

```
export JAVA_HOME=/usr/lib/jvm/java-7-oracle/
export TOMCAT_HOME=/opt/tomcat7

case $1 in
start)
  sh $TOMCAT_HOME/bin/startup.sh
;;
stop)
  sh $TOMCAT_HOME/bin/shutdown.sh
;;
restart)
  sh $TOMCAT_HOME/bin/shutdown.sh
  sh $TOMCAT_HOME/bin/startup.sh
;;
esac
exit 0

```

## 原则与建议

- 容器轻量化。从镜像中产生的容器应该尽量轻量化，能在足够短的时间内停止、销毁、重新生成并替换原来的容器。
- 使用 .gitignore。在大部分情况下，Dockerfile 会和构建所需的文件放在同一个目录中，为了提高构建的性能，应该使用 .gitignore 来过滤掉不需要的文件和目录。
- 为了减少镜像的大小，减少依赖，仅安装需要的软件包。
- 一个容器只做一件事。解耦复杂的应用，分成多个容器，而不是所有东西都放在一个容器内运行。如一个 Python Web 应用，可能需要 Server、DB、Cache、MQ、Log 等几个容器。一个更加极端的说法：One process per container。
- 减少镜像的图层。不要多个 Label、ENV 等标签。
- 对续行的参数按照字母表排序，特别是使用apt-get install -y安装包的时候。
- 使用构建缓存。如果不想使用缓存，可以在构建的时候使用参数--no-cache=true来强制重新生成中间镜像。












