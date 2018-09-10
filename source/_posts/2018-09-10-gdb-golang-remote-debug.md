---
layout: post
title: "使用gdb远程调试Go代码（arm）"
description: ""
category: 技术
tags: [gdb, go]
---

研究的这个最主要原因是arm平台无法使用`delve`工具无法在arm平台上使用，而现实中你的程序不可能没有bug，这时候查起问题来没有debug就太痛苦了~

<!-- more -->

# 编译参数

for Go 1.10 or later: `-gcflags "all=-N -l"`  
for Go 1.9 or earlier: `-gcflags "-N -l"`

于我的环境而言，使用的是如下命令：

```sh
CGO_ENABLED=0 GOOS=linux GOARCH=arm go build -a  -gcflags=all="-N -l" -ldflags '-extldflags "-static"' -o test .
```

# 运行命令

在需要调试的Arm机器上运行如下命令：

```sh
gdbserver :1234 ./test -c ../conf/cfg.linux-arm.json
```

![](https://media-1256569450.cos.ap-chengdu.myqcloud.com/blog/15365650907364.jpg)


# 本地环境

原计划在MacOs环境上运行的，但每次一运行next的时候，gdbserver就会退出，无奈之下还是选择了在docker环境里边进行远程调试。


```sh
docker run --rm -it -v $PWD:/work -v $GOROOT:$GOROOT -v $GOPATH:$GOPATH dockcross/linux-armv7 bash
```

进入docker环境之后，执行 `apt-get install gdb-multiarch cgdb` 安装所需要的`gdb`和`cgdb`命令

然后，我们写一个`.cmds`文件，如`arm-gdb.cmds`: 

```sh
# GOROOT
directory /usr/local/Cellar/go/1.11/libexec
# Remote
target remote 192.168.113.210:1234
# GO runtime和goroutine等支持
source /usr/local/Cellar/go/1.11/libexec/src/runtime/runtime-gdb.py
# 调试的程序文件
file /Users/scue/go/src/path/to/xxx
# 设定断点
b main.main
```

最后运行 `cgdb -d gdb-multiarch -x arm-gdb.cmds` 就可以进行调试了

![](https://media-1256569450.cos.ap-chengdu.myqcloud.com/blog/15365654788203.jpg)

参考链接：（官网）https://golang.org/doc/gdb
