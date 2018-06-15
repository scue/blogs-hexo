---
layout: post
title: "手把手教你编译Mips版本的Busybox"
description: ""
category: 技术
tags: [mips, busybox]
---


# 下载源码

```sh
cd /tmp
wget https://busybox.net/downloads/busybox-1.28.4.tar.bz2
tar jxvf busybox-1.28.4.tar.bz
cd busybox-1.28.4
```

<!-- more -->

# 基础环境

```sh
export ARCH=mips
export SUBARCH=mips
export CROSS_COMPILE=mips-linux-
make defconfig
```

> 提示：确保你的环境上有`mips-linux-gcc`等交叉编译工具，并且在你的`PATH`环境变量里边

由于在编译过程中可能会遇到以下错误：

- `sync.c:(.text+0x130): undefined reference to 'syncfs'`
- `nsenter.c:(.text+0x3f0): undefined reference to 'setns'`

因此，还需要 `make menuconfig` 配置一下：

- `Linux System Utilities` → `nsenter`，去掉该选项
- `Coreutils` → `sync`，去掉该选项

参考链接：https://www.cnblogs.com/softhal/p/5769121.html

# 静态编译

执行 `make -j8 "CFLAGS+=-static"`

![](https://media-1256569450.cos.ap-chengdu.myqcloud.com/blog/15289464456136.jpg)

# 运行效果

![](https://media-1256569450.cos.ap-chengdu.myqcloud.com/blog/15289465295149.jpg)



