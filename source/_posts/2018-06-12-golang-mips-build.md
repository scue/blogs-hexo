---
layout: post
title: "Golang完全静态编译MIPS架构二进制程序"
description: ""
category: 技术
tags: [GO, mips]
---

写一个编译脚本`go-mips-build.sh`：

```sh
#!/bin/bash

export PATH=$PATH:/proj/mtk69527/econet-toolchain/buildroot-2015.08.1/output/host/usr/bin
export AR=mips-linux-ar
export CC=mips-linux-gcc
export CXX=mips-linux-g++

set -x

CGO_ENABLED=0 CC=$CC CXX=$CXX GOOS=linux GOARCH=mips go build -v --ldflags '-linkmode external -extldflags "-static"' "$@"
```

<!-- more -->

假定代码在当前目录下，执行命令编译：`./go-mips-build.sh -o agent .`，效果如下：

![](https://media-1256569450.cos.ap-chengdu.myqcloud.com/blog/15287965610896.jpg)



> 提示：`/proj/mtk69527/econet-toolchain/buildroot-2015.08.1/output/host/usr/bin`是甲方提供的交叉工具链，安装位置必须是这里，有点小奇葩，还好我这边使用Docker，不Care~

