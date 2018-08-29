---
layout: post
title: "GDB远程调试（ARM平台的二进制）"
description: ""
category: 技术
tags: [gdb, arm]
---

最近接到一个使用C++写的二进制程序，运行于Arm平台，文档比较少，于是想到这样子的情况下最简单的理解代码方式就是使用GDB去调试一下，看看实际它都运行得怎么样。

笔者的环境是Mac环境，Mac环境下直接使用gdb远程调试不好配置，于是我的环境主要在docker环境下进行的。

<!-- more -->

# 编译选项

需要在编译的时候加上`-g -O0`选项，才可以方便地进行调试。

假定编译出来的二进制名字叫做`test`

# 远程调试

在docker环境下进行调试

```sh
docker run --rm -it -v $PWD:/work dockcross/linux-armv7 bash
apt-get install gdb-multiarch cgdb
```

> 提示：`/work`是我存放代码的位置

然后在客户端上执行：

```txt
# gdbserver :1234 /tmp/test
Process /tmp/net_detect created; pid = 15229
Listening on port 1234
```

> 提示：这样子就启动了`gdbserver`，监听端口是`1234`

随后在Docker的环境上依次执行：
```
# cgdb -d gdb-multiarch
> target remote 192.168.113.210:1234
> file /work/test
> b main # 对main()函数进行断点
> c
```

> 提示：`192.168.113.210:1234`是我目标机器上`gdbserver`监听的IP和端口


效果展示：

![](https://media-1256569450.cos.ap-chengdu.myqcloud.com/blog/15353679274157.jpg)

接下来，我就可以逐行地看程序的运行到底如何了，结合输出的信息，以便分析其中的逻辑。

# 彩蛋

一些有用的GDB配置

```sh
set watchdog 32000000 # 一些操作的超时时间，比如continue 
set remotetimeout unlimited # 针对gdbserver，Set timeout limit to wait for target to respond 
set print address on # 显示函数参数地址 
set print array on # 数组每一个元素单独一行 
set print pretty on # 结构体每个成员单独一行 
set print union on # 结构体内的联合体元素展开
```
