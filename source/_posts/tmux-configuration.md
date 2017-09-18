---
title: tmux 配置
date: 2017-07-01 13:02:17
tags: [tmux]
categories: [工具, tmux]
---
[前篇](/2017/06/30/tmux/) 主要介绍了 tmux 的基本使用, 这篇主要介绍 tmux 的配置.

## 使用 gpakosz/.tmux

Github 上有个相当不错的 [.tmux 配置](https://github.com/gpakosz/.tmux), 直接安装使用即可.

``` shell
$ cd
$ git clone https://github.com/gpakosz/.tmux.git
$ ln -s -f .tmux/.tmux.conf
$ cp .tmux/.tmux.conf.local .
```

执行完了之后, 重启 tmux 即可(如果此时已经有 tmux 在运行了, 则可以使用 `source-file ~/.tmux` 来更新配置).

效果如下:

![](/images/tmux-configuration_gpakosz.png)

之后, 可以在 `~/.tmux.conf.local` 文件中添加自定义配置.

<!-- more -->

通常, 上面的配置即可满足我们日常的使用并有一个不错的体验, 以下是对 tmux 的配置更详细的解释与说明,
如果对配置的细节感兴趣, 可以继续阅读剩下部分内容.

## .tmux.conf 配置文件

默认的, tmux 从如下位置查找配置文件:

* `/etc/tmux.conf` (系统级)
* `~/.tmux.conf` (用户级)

如果都未找到, 则使用默认的配置. 所以, 一般进行配置, 创建一个用户级别的配置文件即可.

> 注意
>
> 如果是 Mac, 在 `/usr/local/opt/tmux/share/tmux` 文件夹中有示例的配置.

## 常用命令

- set-option
- set-window-option
- unbind

## set-option

设置 tmux 选项. 简短的命令为 `set`. 格式为

`set-option [-aFgoqsuw] [-t target-session | target-window] option value`

如:

`set -g prefix C-a`

表示将 `Ctrl + a` 设置为 `PREFIX`.

其中:

- `-a`: 表示选项需要字符串或者样式(style)时, 值将会被 **添加** 到现有的设置中
- `-F`: 展开选项值中的格式? (expands formats in the option value)
- `-g`: 设置全局的 session 或者窗口选项.
- `-o`: 避免设置一个已经存在的选项
- `-q`: 抑制未知的或者有歧义的选项错误
- `-s`: 设置服务器选项
- `-u`: 复原(unset)选项
- `-w`: 设置窗口选项, 与 `set-window-option` 相同

关于 `set-option` 更详细的秒速以及所有可用的 option 可以 [查看这里](http://man.openbsd.org/OpenBSD-current/man1/tmux.1#set-option)

## 常见问题

### 使用 `PREFIX ,` 重命名窗口后, 执行命令会导致名字发生变化

这个是 tmux 的默认行为, 可以通过添加 `set-option -g allow-rename off` 配置
来防止自动重命名.
