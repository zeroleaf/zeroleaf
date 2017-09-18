---
title: shadowsocks
date: 2017-09-18 22:37:14
tags: [网络]
categories: [工具, 网络]
---

**shadowsocks** 是一个安全的代理网络代理工具.
其分为客户端和服务器两部分, 客户端连接服务器即可通过服务器代理上网.

因此, 如果在国内, 有一台国外的虚拟主机即可代理自由上网;  
而如果在国外, 则可以通过国内的主机代理来绕开版权的限制.

## 服务器

首先确保服务器上安装了 **Python** 与 **pip**.

然后通过 `pip` 安装即可:

``` sh
pip install shadowsocks
```

安装完创建 `/etc/shadowsocks.json` 文件, 内容如下:

``` json
{
    "server":"XXX.XXX.XXX.XXX",
    "server_port":8388,
    "local_port":1080,
    "password":"secret",
    "timeout":600,
    "method":"aes-256-cfb"
}
```

其中:

- `server`: 填入服务器的外网地址即可
- `password`: shadowsocks 连接密码

其他参照如上配置即可.

### 启动服务

创建完配置文件之后, 即可通过如下命令启动后台服务:

``` sh
ssserver -c /etc/shadowsocks.json -d start
```

> 注意
>
> 如果配置文件的路径跟上述的不一样, 注意修改 -c 后的参数


<!-- more -->

## 客户端

客户端安装相对简单, 下载对应的文件即可.
具体请参考 [这里](https://shadowsocks.org/en/download/clients.html).

快捷下载链接:

- [Android](https://github.com/shadowsocks/shadowsocks-android/releases)
- [Win shadowsocks-windows](https://github.com/shadowsocks/shadowsocks-windows/releases)
- [Win shadowsocks-qt5](https://github.com/shadowsocks/shadowsocks-qt5/releases)
- [Mac ShadowsocksX-NG](https://github.com/shadowsocks/ShadowsocksX-NG/releases)

**IOS**

IOS 除了上述的软件之外, 还可以通过 **PP助手** 安装. 具体:

1. 使用 PC 安装 PP助手
2. 在 PP助手 中搜索 `shadowrocket` 安装

### 配置

各种客户端的配置方式都差不多, 根据具体的界面填入如下参数即可:

- 服务器地址
- 服务器端口, 一般 _8388_
- 加密方式, 一般为 _aes-256-cfb_
- 密码

## 附

### 搬瓦工 (bandwagonhost)

比较便宜好用的 VPS 供应商, 博主在用的即是一个位于洛杉矶的年费 $19.99 的 VPS,
刚看了下, 好像有中国到洛杉矶直连的, 这种的速度应该会相对快很多.
更多的 VPS 服务可以参考 [这里](https://bandwagonhost.com/cart.php).

> 注意
>
> 搬瓦工在支付的时候支持优惠码, 因此网上找找相应的优惠码可以有一定的优惠.
