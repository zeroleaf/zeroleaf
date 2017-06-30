---
title: C文件I/O
date: 2017-06-11 10:16:00
tags: [C, I/O]
categories: [C, I/O]
---
IO 是每个程序中基本的操作, 而在大多数的文件I/O操作中, 只需要 5 个函数:
`open`, `read`, `write`, `lseek`, `close` 等. 简单的来说, 几个基本函数是系统调用, 
提供了文件IO的最原始的操作.

## 文件描述符

对内核而言, 所有打开的文件资源 **都是通过文件描述符** 引用. 文件描述符是一个 **非负整数**.
在打开/创建文件时, 内核会向进程返回对应的文件描述符, 而其也是 `read`, `write` 等的参数.

### 特殊的文件描述符

-  STDIN_FILENO(0): 标准输入
- STDOUT_FILENO(1): 标准输出
- STDERR_FILENO(2): 标准错误

值得注意的是, 虽然 0, 1, 2 已被标准化, 但在编码的时候最好还是使用 XX_FILENO 预定义常量.
常量预定义在 `<unistd.h>` 中.

<!-- more -->

## open

打开或创建一个文件, 函数原型如下:

``` C
# 成功: 返回最小未使用的文件描述符数值
# 失败: -1

#include <fcntl.h>

int open(const char *path, int oflag, ...);

int openat(int fd, const char *path, int oflag, ...);
```

_path_ 参数为要打开或创建的文件的名称. flags 为该函数的选项, 一般为一个或
多个常量进行 "按位或" 运算构成, 可选值如下:

| 常量        | 说明                                                                                                     |
|-------------|----------------------------------------------------------------------------------------------------------|
| O_RDONLY    | 只读打开                                                                                                 |
| O_WRONLY    | 只写打开                                                                                                 |
| O_FDWR      | 读写打开                                                                                                 |
| O_APPEND    | 以追加模式打开该文件                                                                                     |
| O_CLOEXEC   | 在文件描述符上设置 FD_CLOEXEC 常量                                                                       |
| O_CREATE    | 如果文件不存在, 则创建                                                                                   |
| O_DIRECTORY | 如果 _path_ 不是一个文件夹, 则调用失败                                                                   |
| O_EXCL      | 如果同时设置了 O_CREAT 并且文件存在则出错                                                                |
| O_NOCTTY    | 如果 _path_ 引用的是终端设备, 则不将该设备分配作为此进程的控制终端                                       |
| O_NOFOLLOW  | 如果 _path_ 引用的是一个符号链接, 则出错                                                                 |
| O_NONBLOCK  | 非阻塞, 不等待数据可用                                                                                   |
| O_SYNC      | 使每次 write 等待物理 I/O 操作完成, 包括由该 write 操作引起的文件属性更新所需的 I/0                      |
| O_TRUNC     | 以写模式成功打开, 将文件长度截断为 0                                                                     |
| O_DSYNC     | 使每次 write 要等待物理 I/O 操作完成, 但是如果该写操作并不影响读取刚写入的数据, 则不需等待文件属性被更新 |
| O_RSYNC     | 使每一个以文件描述符作为参数进行的 read 操作等待, 直至所有对文件同一部分挂起的写操作都完成               |

其中, 前 3 者为打开模式, 必须并且只能设置一种, 余下的为附加选项, 可以通过 按位或 运算添加.
更多详细的描述, 可通过 `man 3 open` 来查看.

除非 _path_ 参数是一个相对路径, 否则 `openat` 与 `open` 调用的行为是完全一致的.
在这种情况下(_path_ 为相对路径), 待打开的文件所在的目录为 fd 文件所在的目录而不是当前目录;
如果 fd 为特殊值 AT_FDCWD, 则使用当前目录此时行为与 `open` 也是完全一致.

## creat

创建一个新文件, 函数原型:

``` C
#include <fcntl.h>

int creat(const char *path, mode_t mode);
```

当前的实现与 `open(path, O_CREAT | O_TRUNC | O_WRONLY, mode);` 相同

## close

关闭一个打开的文件, 函数原型:

``` C
# 成功: 返回 0
# 失败: 返回 -1

#include <unistd.h>

int close(int fildes);
```

注意

- 关闭一个文件时还会释放该进程加在该文件上的所有记录锁.
- 当一个进程终止时, 内核自动关闭它所有打开的文件. 
  <u>(很多程序都利用了这一功能而不显式地用 close 函数关闭打开文件)</u>

## lseek

表征读/写文件的便宜量. 除非在打开文件时特别指定 **O_APPEND** 选项, 否则该偏移量都为 **0**.

可以通过该调用显式地设置一个打开文件的偏移量:

``` C
# 成功: 新的文件偏移量
# 失败: -1

#include <unistd.h>

off_t lseek(int fildes, off_t offset, int whence);
```

对参数 _offset_ 的解释与 _whence_ 有关:

- 若 _whence_ 是 **SEEK_SET**, 则将该文件的偏移量设置为距文件开始处 _offset_ 个字节.
- 若 _whence_ 是 **SEEK_CUR**, 则将该文件的偏移量设置为当前值加 _offset_, _offset_ 可为正或负.
- 若 _whence_ 是 **SEEK_END**, 则将该文件的偏移量设置为文件长度加 _offset_, _offset_ 可为正或负. 

其中

> **off_t** 其实是一个整型类型, 根据系统的不同实际值也不一样, 但至少不小于 **int**  

注意

> 文件偏移量可以大于当前文件长度, 在这种情况下, 对该文件的下一次写将加长该文件, 
> 并在文件中构成一个空洞, 位于文件中但没有写过的字节都被读为 **0**

## read

从打开的文件中读取数据, 函数原型:

``` C
# 尝试从 fd 引用的文件中读取 nbytes 字节的数据到 buf 中
# 成功: 返回读到的字节数, 如果已到达文件尾端, 则返回 0
# 失败: 返回 -1

#include <unistd.h>

sszie_t read(int fd, void *buf, size_t nbytes);
```

以下情况可使实际读到的字节数少于要求的字节数:

- 读取普通文件时, 在读到要求字数之前已经到达了文件尾端
- 当从终端设备读时, 通常一次最多读取一行
- 当从网络读取时, 网络中的缓冲机制可能造成返回值小于所要求读的字节数
- 当从管道或FIFO读取, 如果管道包含的字节少于所需的数量, 那么 read 将只返回实际可用的字节数
- 当从某些面向记录的设备读时, 一次最多返回一个记录
- 当一信号造成中断, 而已经读了部分数据量时

## write

向打开的文件写入数据, 函数原型:

``` C
# 尝试将 buf 中的 nbytes 字节数据写入到文件中
# 成功: 返回已写的字节数
# 失败: 返回 -1
#include <unistd.h>

ssize_t write(int fd, const void *buf, size_t nbytes);
```

其返回值通常与参数 nbytes 值相同, 否则表示出错.

对于普通文件, 写操作从文件的当前偏移出开始. 如果在打开文件时, 指定了 **O_APPEND** 选项, 
则在每次写操作之前, 将文件偏移量设置在文件的当前结尾处. 在一次成功写之后, 该文件偏移量增加实际写的字节数.

## 示例

``` C
#include <fcntl.h>
#include <unistd.h>
#include <string.h>
#include <assert.h>

#define BUF_SIZE 1024

int main(void)
{
    char *content  = "Nice to meet you!";
    char *filename = "io.txt";

    // Write message to file io.txt
    int fd             = open(filename, O_WRONLY | O_CREAT, 0644);
    write(fd, content, strlen(content));
    close(fd);

    // Read data from io.txt
    char    buf[BUF_SIZE];
    ssize_t read_count = 0;

    // Skip word Nice
    fd = open(filename, O_RDONLY);
    off_t offset = lseek(fd, (off_t) strlen("Nice") + 1, SEEK_SET);
    read(fd, buf, BUF_SIZE);
    close(fd);

    assert(strcmp("to meet you!", buf) == 0);
}
```

## 文件共享



## 参考

- UNIX 环境高级编程(第三版)