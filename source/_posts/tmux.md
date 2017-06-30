---
title: tmux
date: 2017-06-30 10:00:21
tags: [tmux]
---
> tmux是一个优秀的终端复用软件, 使用它最直观的好处就是，通过一个终端登录远程主机并运行tmux后，
> 在其中可以开启多个控制台而无需再“浪费”多余的终端来连接这台远程主机；当然其功能远不止于此。
>
> ---- 摘自百度百科

## 安装

### Mac

mac 可以通过 brew 来安装:

`brew install tmux`

## 基本使用

tmux 主要包含如下几个部分的功能:

* 会话 (session)
* 窗口 (window)
* 面板 (panel)

可以通过 `PREFIX ?` 来列出所有预定义的快捷键以及其绑定的命令

## 会话

### 创建

通过 `tmux` 命令即可开始一个会话, 默认的该会话的名字为 0, 如果想创建一个指定名字的会话, 则可通过

`tmux new -s <session_name>`

命令来创建一个会话.

> 注意
>
> 本质上来说, 如果只键入 tmux, 其功能应该是打开最近一次的会话, 如果会话不存在,
> 则创建一个新的会话. 因此, 在已经有一个会话的情况下, 通过 tmux 并不能创建一个新的会话出来,
> 此时, 必须使用命名的方式来创建.
>
> 另外, 新建的命令需要在实际终端中执行(即, 如果当前在 tmux session, 需要先分离),
> 不然会报 'sessions should be nested with care, unset $TMUX to force'.

### 分离

要将一个会话放到后台(并不是停止/退出), 可以通过 detach 命令来完成, 即

`tmux detach`

也可以通过快捷键 `Ctrl-b d` (即同时按住 Control + b, 然后松开再按 d).
同时, `Ctrl-b` 为 tmux 默认的前缀修饰组合键, 之后将用 `PREFIX` 来表示.

### 重入

在分离之后, 如果是要进入最近一次分离的会话, 则简单的通过 `tmux` 即可.
如果是重入到另外一个会话, 则可以通过 `tmux attach -t <session_name>` 来完成.

如果当前已经在会话中了, 执行 attach 命令也会遇到之前创建时的同样操作,
这时候应该使用 switch-client 命令, 即 `tmux switchc -t <session_name>` 来完成.

### 关闭/结束会话

如果当前在会话中, 则通过 exit 命令即可结束当前会话.

如果当前在终端, 则可以通过 `tmux kill-session -t <session_name>` 来完成.

### 列表

通过 `tmux list-sessions` 可以列出当前所有的会话列表, 该命令在会话跟终端都可以使用.

## 窗口

一个会话中可以包含多个窗口, 窗口是占满整个终端的 (可以类比 iTerm 中的 Tab).

* `PREFIX c`: 创建
* `PREFIX ,`: 重命名
* `PREFIX n`: 移动到下一个窗口
* `PREFIX p`: 移动到上一个窗口
* `PREFIX <num>`: 其中, num 是窗口对应的编号, 通过该命令可以快速跳转到对应的窗口
* `PREFIX f`: 通过名字来查找窗口
* `PREFIX w`: 窗口列表, 可以选择
* `PREFIX &`: 结束当前窗口, 在结束之前会有一个确认消息提示
* `exit`: 直接关闭/结束当前窗口

## 面板

面板相对窗口来说更轻量一些, 它是在窗口上的, 我们可以将一个窗口分成几个面板.

* `PREFIX %`: 将当前窗口垂直平分, 并将焦点移到新建的面板
* `PREFIX "`: 将当前窗口水平评分, 并将焦点移到新建的面板
* `PREFIX o`: 在面板之间循环移动
* `PREFIX Up`: 移动到上一个面板
* `PREFIX Down`: 移动到下一个面板
* `PREFIX Left`: 移动到左边的面板
* `PREFIX Right`: 移动到右边的面板
* `PREFIX SPACEBAR`: 在预定义的面板布局之间循环切换. 注意, 自定义的切换之后不能简单的撤销
* `PREFIX X`: 关闭当前面板, 在关闭之前会有一个确认消息提示

## 命令模式

* `PREFIX :`: 进入命令模式, 在这里可以执行 tmux 的命令

## 常见问题

### iTerm 中垂直分割线无法显示

在 `iTerm->Preferences->Profiles->Text` 中取消
`Treat ambiguous-width characters as double width` 的勾选.

详见: https://stackoverflow.com/questions/9821973/tmux-borders-are-drawn-with-dashed-lines-how-can-i-change-them-to-continuous-li
