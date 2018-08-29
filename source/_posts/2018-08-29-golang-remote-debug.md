---
layout: post
title: "Go语言远程调试方法"
description: ""
category: 技术
tags: [go, remote, debug]
---

基本的环境：

- 本地是MacOS和IntelliJ Idea环境
- 远程是Linux-amd64的环境

简而言之，在本地的IDEA开发环境里边，对远端运行的程序进行调试。

<!-- more -->

# 编译选项

- 对于golang 1.10及以上，需要加上`-gcflags "all=-N -l"`
- 对于golang 1.9及以下，需要加上`-gcflags "-N -l"`

简单来说，我一般这样编译：

```sh
CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -a -gcflags "all=-N -l" -ldflags '-extldflags "-static"' -o main ./cmd/serve/main.go
```

这样子既可以完全静态编译，又加上了调试信息。

# 远端运行

> 假定程序名是`main`

如果程序已启动：

```sh
dlv --listen=:2345 --headless=true --api-version=2 attach $(pidof main)
```

如果程序未启动：

```sh
dlv --listen=:2345 --headless=true --api-version=2 exec ./main
```

> 提示: 关于`dlv`，查看官网：https://github.com/derekparker/delve ，一般我喜欢在本地将它静态编译为`dlv-linux-amd64`，然后拷贝到远端的`/usr/local/bin/dlv`，这样子就可以直接使用了。

> 提示：记得将`2345`端口开放给指定的IP可以访问，这样子你本地才可以连接至这个端口进行调试。

# 本地配置

在IDEA环境上，点击`Edit Configurations...`，按如下进行配置：

![](https://media-1256569450.cos.ap-chengdu.myqcloud.com/blog/15355151412231.jpg)

点击Debug的按钮

![](https://media-1256569450.cos.ap-chengdu.myqcloud.com/blog/15355152179832.jpg)

然后在你的代码上设定断点就可以进行调试了：

![](https://media-1256569450.cos.ap-chengdu.myqcloud.com/blog/15355153762364.jpg)

一旦连接上了，红色框框内的这个按钮，会从灰色变成了彩色，表示可以使用了，然后就可以进行调试了，非常的方便~
