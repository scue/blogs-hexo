---
layout: post
title: "手把手教你如何交叉编译MTR二进制程序"
description: ""
category: 技术
tags: [C/C++, MTR]
---



# 相关背景

其实我也不知道MTR是干嘛使用的，不过我大致知道它是一个网络诊断工具，源码是使用C/C++编写的。

它的源码位于`https://github.com/traviscross/mtr`。

我们去官网看看README.md，大致长这样子的：

<!-- more -->

![](https://ws1.sinaimg.cn/large/6e22ca27gy1fqjz5fj41xj20u80jaq88)


Don't panic，一切都是如此简单，我们只需要一个Docker环境就，配合使用`dockcross/linux-armv7`可以进行交叉编译了。

** 你有疑问？**

1. 关于`Docker`：可以到官网 https://www.docker.com/get-docker 下载来安装试一下。

2. 关于`dockcross/linux-armv7`镜像：你可以从这里（`https://github.com/dockcross/dockcross`）获取更多信息，这里就不再重复了。

# 源码下载


```sh
git clone https://github.com/traviscross/mtr
```

![](https://ws1.sinaimg.cn/large/6e22ca27gy1fqjz5hcnkwj20lm0dqtct)


# 进入交叉编译环境

如果你已经安装好了docker环境，直接运行以下命令（若找不到镜像它会自动下载）

```sh
docker run --rm -it -v $PWD/mtr:/work dockcross/linux-armv7 bash
```

# 编译ARM版本的二进制文件

```sh
./bootstrap.sh && ./configure && make && make install
```

![](https://ws1.sinaimg.cn/large/6e22ca27gy1fqjz5hcqitj20o10ieteo)



我们检查一下它安装的二进制文件：

![](https://ws1.sinaimg.cn/large/6e22ca27gy1fqjz5h7xrij20o10ie79w)


发现安装的是`mtr`和`mtr-packet`文件（当然，这只是安装到你的Docker容器里边，与宿主机没有一毛钱关系）。

# 确认一下依赖库

有一些二进制运行的时候，有依赖库文件，检查二进制依赖哪些库，可以使用`ldd`命令 ，如下：

```sh
# ldd /bin/grep
 linux-vdso.so.1 (0x00007ffcdb99c000)
 libpcre.so.3 => /lib/x86_64-linux-gnu/libpcre.so.3 (0x00007ff9c907b000)
 libdl.so.2 => /lib/x86_64-linux-gnu/libdl.so.2 (0x00007ff9c8e77000)
 libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007ff9c8acc000)
 libpthread.so.0 => /lib/x86_64-linux-gnu/libpthread.so.0 (0x00007ff9c88af000)
 /lib64/ld-linux-x86-64.so.2 (0x00007ff9c951c000)
```

但是，ldd去检查ARM版本的二进制程序的时候就不太正常，如下：

```sh
# ldd mtr
 not a dynamic executable
# file mtr
mtr: ELF 32-bit LSB executable, ARM, EABI5 version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux-armhf.so.3, for GNU/Linux 2.6.32, BuildID[sha1]=b98457bce3813c13f0bdf346cbdb21e023e395b5, not stripped
```

因此，我们换一种方式，使用`arm-linux-gnueabihf-readelf`去检查就好了：

```sh
# arm-linux-gnueabihf-readelf -a mtr | grep Shared
 0x00000001 (NEEDED) Shared library: [libresolv.so.2]
 0x00000001 (NEEDED) Shared library: [libncurses.so.5]
 0x00000001 (NEEDED) Shared library: [libtinfo.so.5]
 0x00000001 (NEEDED) Shared library: [libm.so.6]
 0x00000001 (NEEDED) Shared library: [libc.so.6]
```

# 检查哪些依赖库不存在

一般而言，玩客云上边的动态库运行的时候，通常可能是从以下位置加载的：

```txt
/lib
/lib32
/libexec
/usr/lib
/usr/lib32
/usr/libexec
/app/system/usr/lib
/thunder/lib
```

其中的`/thunder/lib`是一般都是手工在脚本里边去添加的`LD_LIBRARY_PATH`变量里边去的。

我们可以通过一个脚本去检查这些动态库是否存在，比如我写了一个简单的检查脚本，位于 https://gist.github.com/scue/c8879958ea76dcbc1463eda4ada4e768 

通过检查得到，发现libtinfo.so是查找不到的，如下：

![](https://ws1.sinaimg.cn/large/6e22ca27gy1fqjz5egjd4j20ht06c757)


# 一种解决方式

![](https://ws1.sinaimg.cn/large/6e22ca27gy1fqjz5gv17tj20rc0l8wpw)


解决方式有多种，一种是自己去下载libtinfo相关源码，然后自己编译，另一种是可以像我这样子直接看看`/app`目录之下是否有人已经编译出来了。

总结起来就是如下几行命令：

```sh
find /app -name libtinfo.so\* # 找到动态库
cp /app/docker/aufs/diff/289eedd8bcde52c970e6a6439faa943fed1fe03629e4e0667402325c52026af0/lib/arm-linux-gnueabi/libtinfo.so.5 /tmp/lib/
export TERM=xterm
/tmp/mtr # 运行程序
```

![](https://ws1.sinaimg.cn/large/6e22ca27gy1fqjz5f4isjj20r503sq3t)


All Done，如有问题，重新阅读一遍文章，保存此文章可以当作手册来使用。`(⊙o⊙)`


