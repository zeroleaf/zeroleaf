---
title: Docker 基础
date: 2017-09-21 00:34:51
tags: [Docker]
categories: [Docker]
---

该文章主要介绍 Docker 相关的基础内容, 主要包括:

- 基本概念
- 常用操作


## 基本概念

> **Docker** 是一个开放源代码软件项目，让应用程序布署在软件容器下的工作可以自动化进行，
> 借此在Linux操作系统上，提供一个额外的软件抽象层，以及操作系统层虚拟化的自动管理机制。
>
> -- 维基百科

Docker 主要包含有如下的核心组件:

- Docker 客户端和服务器
- Docker 镜像
- Docker 容器
- Registry

Docker 总架构图如下(图片来自 infoq):

![Docker 总架构图](http://cdn1.infoqstatic.com/statics_s1_20170919-0429u7/resource/articles/docker-source-code-analysis-part1/zh/resources/001.jpg)

<!-- more -->


## 客户端服务器

从上述的 Docker 总架构图中可以很容易的看出 Docker 是一个客户端, 服务器架构.

Docker 以 **root** 权限运行它的守护进程, 来处理普通用户无法完成的操作 (如挂载文件系统).
守护进程启动后, 会监听 _/var/run/docker.sock_  这个 Unix 套接字文件, 以处理来自客户端的 docker 请求.

该套接字的权限如下:

``` sh
srw-rw---- 1 root docker 0 Oct  1 11:11 /var/run/docker.sock
```
如果系统中存在 _docker_ 用户组(Ubuntu 在安装 docker 的时候默认会添加) 则守护进程会将套接字的
用户组设置为 _docker_. 因此, 将当前用户加入 _docker_ 用户组即可不用 `sudo` 来执行命令:

``` sh
sudo usermod -aG docker $USER
```

> 注意: 如果使用的是 **Docker for Mac**, 则上述的套接字文件路径为 **/private/var/run/docker.sock**

> 注意: 当前(2017.09)使用的是 _docker-ce_ 版本, 其 docker 守护进程的启动方式已由原来的
  `docker -d -H unix://` 方式改为了用 `dockerd -H` 的形式启动, 通过 `ps` 查看的结果如下:
  ``` sh
  root      1081  0.1  4.3 503396 44396 ?        Ssl  11:12   0:01 /usr/bin/dockerd -H fd://
  ```

## Docker 镜像

Docker 的镜像相当于一个最小的系统, 它是用来用来创建容器的模板, 每次从镜像中创建的新容器,
其初始环境都是一样的, 同时也可以通过命令行参数或者 **Dockerfile** 对新创建的容器进行定制,
这就保证了镜像的通用性, 以及对于程序环境的一致性.

可以通过 `docker pull` 命令从远端仓库拉取一个镜像, 如可以通过如下命令获取 ubuntu 17.04 版镜像:

``` sh
docker pull ubuntu:17.04
```

其中 17.04 为标签(Tag)名称, 如果省略, 默认为 **latest**, 对于 ubuntu 而言, latest 指向的是最新的 LTS 版本,
因此当前为 16.04.

在拉取了一个镜像之后, 可以通过 `docker images` 命令来查看当前已拉取的镜像:

``` sh
docker images

# 输出如下

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ubuntu              17.04               6ca5545cc1ef        12 days ago         94.7MB
```

其中, **IMAGE ID** 是容器的标识, 在对容器进行操作的时候需要使用到该标识.

如果想删除镜像, 可以先通过上述命令找到镜像的 ID, 然后通过 `docker rmi` 命令删除即可:

``` sh
docker images
# docker images 输出
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ubuntu              17.04               6ca5545cc1ef        12 days ago         94.7MB
nginx               latest              da5939581ac8        2 weeks ago         108MB

# 删除 nginx 镜像
docker rmi da5939581ac8

# 查看当前所有的镜像
docker images
# 当前所有镜像
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ubuntu              17.04               6ca5545cc1ef        12 days ago         94.7MB
```

> 值得注意的是:
>
  - 当有容器在运行的时候, 是不能删除镜像的;
  - 当镜像对应的容器全部都停止时, 可以通过 **-f** 选项来强制删除镜像, 但不会同时删除已存在的容器;
    由此可见, 当容器创建完之后, 它就是个独立的存在, 对镜像的更改不会影响到现有的容器.
  - 可以安全的删除未被使用的镜像

> `docker images` 命令的 `-q` 选项只显示容器的标识, 因此可以通过如下命令来移除所有镜像:
>
  ``` ssh
  docker rmi $(docker images -q)
  ```

## Docker 容器

容器是镜像运行的实例. 类比面向对象编程语言, 镜像相当于类, 而容器则相当于类的一个实例.

### 创建一个新的容器

可以通过 `docker run` 命令来创建一个新的容器:

``` sh
docker run -it --name demo1 ubuntu:17.04
```

上述命令即以 ubuntu 的 17.04 版本为基准, 构建了一个新的容器实例. 其中, 通过 `--name` 参数制定该容器的名字为 demo1,
`-it` 来模拟一个 tty 终端进入到容器中与之交互. `docker run` 命令还有其他的参数, 可以通过 `docker run --help` 来查看.

### 查看容器的运行状态

在上一步, 我们已经创建并启动了一个容器实例, 当前终端显示如下:

``` sh
root@efbda911a3ac:/#
```

说明我们已经以 root 用户登录该容器了. 其中, efbda911a3ac 为容器的标识.
可以通过 `docker ps` 命令查看容器的运行状态, 打开另一个终端窗口(一般 Cmd+t 即可), 执行如下命令:

``` sh
docker ps

# 输出
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
efbda911a3ac        ubuntu:17.04        "/bin/bash"         9 minutes ago       Up 9 minutes                            demo1
```

可以看到之前我们创建的 demo1 容器确实在运行.

切回容器所在的终端窗口, 执行 `exit` 命令以退出与容器的交互. 此时在执行 `docker ps` 命令则没有响应容器的信息输出,
因为 `docker ps` 只显示当前运行中的容器, 可以通过 `-a` 选项来显示所有的容器, 即:

``` sh
docker ps -a

#输出
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                     PORTS               NAMES
efbda911a3ac        ubuntu:17.04        "/bin/bash"         17 minutes ago      Exited (0) 3 minutes ago                       demo1
```

则可以清楚的看到 demo1 的状态以及其退出的时间. 同时, 根据我们也知道了, 如果停止与容器交互, 容器也会停止运行.
事实上, 容器是在上述 COMMAND 执行期间才处于运行状态的, 而当 COMMAND 执行完之后, 容器就相应的退出了.
这也是为什么我们 `exit` 之后容器停止的原因, 因为 `/bin/bash` 命令执行完成了.

### 启动现有的容器

值得注意的是, `docker run` 命令是每次都新 **创建** 一个容器. 因此如果想要启动现有的容器, 应该使用 `docker start` 命令:

``` sh
docker start -ai demo1
```

则我们重新进入了 demo1 容器的模拟终端. `-ai` 选项即是制定与该容器交互.
此时在通过 `docker ps` 即可看到 demo1 容器已经处在运行状态.

### 在容器内部运行进程

对上述启动操作, 如果没有指定 `-ai` 选项, 则容器是放在后台运行的.
此时, 我们可以使用 `docker attach` 命令来打开与该容器交互的模拟终端:

``` sh
docker attach demo1
```

同时, 对于在后台运行的容器, 可以使用 `docker exec` 命令来与之交互, 如:

``` sh
# 创建
docker exec -d demo1 touch /etc/new_config_file

# 查看
docker exec -t demo1 ls -al /etc/new_config_file
# 输出
-rw-r--r-- 1 root root 0 Oct  2 13:46 /etc/new_config_file
```

上述命令都是在交互式的对容器进行操作, `-d` 指定容器以后台模式运行, `-t` 生成一个模拟 tty 终端以显示容器的输出.

同时, 也可以通过如下命令达到与 attach 一样的与容器直接交互的效果:

``` sh
docker exec -it demo1 /bin/bash
```

### 停止容器

要想停止容器, 除了 `exit` 之外, 也可以使用 `docker stop` 命令:

``` sh
docker stop demo1
```

### 重命名容器

在上述操作中, 我们使用了该容器的名字 _demo1_ 而不是标识来启动该容器, 说明给容器起一个有意义的名字, 在后续操作会带来很大的便利.
可以通过 `docker rename` 命令来对容器重命名, 但要注意必须保证名字的唯一性:

``` sh
# 将 demo1 重命名为 c1
docker rename demo1 c1

# 查看容器状态
docker ps -a
# 输出
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                     PORTS               NAMES
efbda911a3ac        ubuntu:17.04        "/bin/bash"         35 minutes ago      Exited (0) 3 minutes ago                       c1
```

其中, _demo1_ 即容器部分, 可以是当前容器的名字, 也可以是容器的标识.

### 查看容器日志

通过 `docker logs` 可以查看容器内的输出(标准输出), 这能很方便的查看放在后台运行的容器的输出.
可以通过 `-f` 选项来指定跟随输出, `-t` 选项显示日志的时间. 更多可通过 `docker logs --help` 来查看.

### 查看容器内的进程

可以通过 `docker top` 命令来查看当前容器内的进程:

``` sh
docker top demo1

# 输出
PID                 USER                TIME                COMMAND
3958                root                0:00                /bin/bash
```

### 查看容器完整信息

可以使用 `docker inspect` 命令查看容器完整的信息:

``` sh
docker inspect demo1

#输出
[
    {
        "Id": "f3e847b058c1340b79be2cf86a427f31fcd5185d335599718ab8e7a54481be77",
        "Created": "2017-10-02T13:42:37.8105815Z",
        "Path": "/bin/bash",
        "Args": [],
        ...
    }
]
```

### 删除容器

可以通过 `docker rm` 命令来删除容器.

```
docker rm c1
# 输出
Error response from daemon: You cannot remove a running container efbda911a3ac1829c30399859e0df26bc53096129796dc39bf6985f913bc721f. Stop the container before attempting removal or force remove

docker rm -f c1
# 输出
c1
```

通过该操作可知, rm 不能移除运行中的容器, 但是可以通过附加 `-f` 选项来强制移除运行中的容器.
同时在删除容器的时候还可以移除制定的 link 与关联的 Volumn, 更多操作可以通过 `docker rm --help` 来查看.

> **TIP**
>
  与 images 命令相同, ps 命令也有一个 -q 选项仅显示容器的 ID, 可以通过该选项来一次性删除全部的容器:
>
  ``` sh
  docker rm $(docker ps -aq)
  ```


## Registry

Docker仓库是镜像的集中管理地, 默认的为 Docker Hub.

### 仓库类型

Docker Hub 中仓库分为两种: 用户仓库和顶层仓库. 用户仓库是由用户创建的, 而顶层仓库是由 Docker 管理的.
用户仓库由两部分组成:

- 用户名
- 仓库名

与之相对的, 顶层仓库则只包含仓库名, 顶层仓库中的镜像是架构良好, 最新且安全的.

### Docker Hub 加速

国内访问 Docker Hub 速度比较慢, 可以通过通过配置加速器来加速该过程, 具体参见如下相应的教程:

- [DaoCloud](https://www.daocloud.io/mirror)


## 参考

- [Docker源码分析（一）：Docker架构](http://www.infoq.com/cn/articles/docker-source-code-analysis-part1)
- [Get Docker CE for Ubuntu](https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu/)
