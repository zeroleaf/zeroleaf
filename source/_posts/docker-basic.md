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

### 客户端服务器

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

### Docker 镜像

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

### Docker 容器

TBD

## 参考

- [Docker源码分析（一）：Docker架构](http://www.infoq.com/cn/articles/docker-source-code-analysis-part1)
- [Get Docker CE for Ubuntu](https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu/)
