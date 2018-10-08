---
layout: post
title: "如何交叉编译ipfs并运行于android平台上"
description: ""
category: 技术
tags: [ipfs, android]
---

> IPFS is the Distributed Web
A peer-to-peer hypermedia protocol
to make the web faster, safer, and more open.

<!-- more -->

# 编译过程

## Proxy环境 

由于`https://ipfs.io/`站点无法访问，因此需要设定一下Proxy环境变量

```sh
export http_proxy=http://127.0.0.1:1087/
export https_proxy=http://127.0.0.1:1087/
export HTTP_PROXY=http://127.0.0.1:1087/
export HTTPS_PROXY=http://127.0.0.1:1087/
```

> 提示：`127.0.0.1:1087`是我使用[shadowsocksx-ng](https://github.com/shadowsocks/ShadowsocksX-NG)开启的一个本地代理服务。

或者，你可以使用这个host配置：

```txt
217.182.195.23 ipfs.io
```

## 下载源码

下载比较简单：

```sh
go get -u -v github.com/ipfs/go-ipfs
```

## 尝试编译

看了下makefile文件，发现可以通过`make build`进行编译，但结果是一直卡在gx那里

![](https://media-1256569450.cos.ap-chengdu.myqcloud.com/blog/15381932007559.jpg)

其实是`gx`命令无法从环境变量上获取Proxy代理，新版本的`gx`已经解决了这个问题。

> 提示：通过`make build`下载的`gx`和`gx-go`命令是链接至`gx-v0.13.0`和`gx-go-v1.7.0`，它们都位于`github.com/ipfs/go-ipfs/bin/`目录下。


## 替换gx和gx-go命令

我们下载新版本的gx和gx-go

```sh
go install github.com/whyrusleeping/gx 
go install github.com/whyrusleeping/gx-go
```

然后进入 `bin`目录，将`gx-v0.13.0`和`gx-go-v1.7.0`链接至新版本的位置即可

```txt
$ tree bin
bin
├── tmp/
...
├── gx -> gx-v0.13.0*
├── gx-go -> gx-go-v1.7.0*
├── gx-go-v1.7.0 -> /Users/scue/go/bin/gx-go*
├── gx-v0.13.0 -> /Users/scue/go/bin/gx*
...
```

## 重新编译

再次执行`make build`，你会发现一切都很顺利。

```sh
$ file cmd/ipfs/ipfs
cmd/ipfs/ipfs: Mach-O 64-bit executable x86_64
```

## 适配Android

适配Android你必须得拥有NDK编译工具链(请参考：[独立工具链](https://developer.android.com/ndk/guides/standalone_toolchain?hl=zh-cn))，并设定以下环境变量：

```sh
export ANDROID_NDK=/Users/scue/source/Android/android-ndk-r16b
export TOOLCHAINS=/Users/scue/source/Android/arm64-android-toolchain-target24
export CC=${TOOLCHAINS}/bin/aarch64-linux-android-clang
export CXX=${TOOLCHAINS}/bin/aarch64-linux-android-clang++
export GOOS="android"
export GOARCH="arm64"
export CGO_CFLAGS="-target aarch64-none-linux-android --sysroot ${TOOLCHAINS}/sysroot"
export CGO_CPPFLAGS="-target aarch64-none-linux-android --sysroot ${TOOLCHAINS}/sysroot"
export CGO_LDFLAGS="-target aarch64-none-linux-android --sysroot ${TOOLCHAINS}/sysroot"
export CGO_ENABLED=1
HOST_EXTRA_LDFLAGS="-pie -fPIE -fPIC"
```
然后再`make build`

```sh
$ file ./cmd/ipfs/ipfs
./cmd/ipfs/ipfs: ELF 64-bit LSB shared object, ARM aarch64, version 1 (SYSV), dynamically linked, interpreter /system/bin/linker64, not stripped
```

# 在Android上运行起来

```
adb push cmd/ipfs/ipfs /data/local/tmp/
adb shell 
> export IPFS_PATH=/data/local/tmp/.ipfs/
> cd /data/local/tmp/
> ./ipfs init
> ./ipfs cat /ipfs/QmS4ustL54uo8FzR9455qaxZwuMiUhyvMcX9Ba8nUH4uVv/readme
```

# 在Android运行Go程序可能会遇到到的问题

1. `getrandom`系统调用不正确，导致出现一堆内核堆栈错误
2. 时区问题，默认是UTC时间

> 这些问题的解决方法都有在我的博客里搜索得到。

# 参考链接

1. [IPFS编译](https://www.jianshu.com/p/363749153792)
2. [I am unable to set CFLAGS and LDFLAGS when building go](https://github.com/golang/go/issues/1234#issuecomment-66053090)
3. [github.com/ipfs/ipfs](https://github.com/ipfs/ipfs)
4. [分布式文件系统 IPFS 与 FileCoin](https://draveness.me/ipfs-filecoin)
