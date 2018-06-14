---
layout: post
title: "GO语言混合C语言链接外部的库"
description: ""
category: 技术
tags: [GO, CGO]
---

与甲方合作，对方提供的是`libcapi.so`动态库，而我期望我的程序依然<font color="#FF4500">我(bian)行(ti)我(lin)素(shang)</font>地使用GO语言进行开发。

OK，撸起裙子加油干！

<!-- more -->

# 相关代码

在我的GO语言程序上是这样子的：

```go
// #include "stdlib.h"
// #include "dbus_service.h"
// #cgo LDFLAGS: ${SRCDIR}/libdbus_service.a
// #cgo LDFLAGS: -L /root/openwrt_cc/staging_dir/target-mips-mtk-linux-uclibc/usr/lib
// #cgo LDFLAGS: -lcapi -lglib-2.0 -lgobject-2.0 -lgmodule-2.0 -lgio-2.0 -lffi -lpthread -lz
import "C"
// import ... // 其他的就不填写了

func main() {
	// 隐藏业务相关代码
	
	appName := C.CString("datacollect")
	C.dbus_init(appName)  // 初始化
	defer C.free(unsafe.Pointer(appName)) // 一定要释放指针，避免内存泄露！
	defer C.dbus_deinit() // 反初始化
	select {} // 程序不会退出，由其他go routine在后台工作
}
```

头文件：`dbus_service.h`

```c
int dbus_init(const char *appname);
int dbus_deinit();
```

编译出来的静态库文件：`libdbus_service.a`

> 提示：编译成动态库也是OK的，随你喜欢。~

# 编译方法

看看`go-mips-build.sh`文件：

```sh
#!/bin/bash

export PATH=$PATH:/proj/mtk69527/econet-toolchain/buildroot-2015.08.1/output/host/usr/bin
export AR=mips-linux-ar
export CC=mips-linux-gcc
export CXX=mips-linux-g++

set -x

CGO_ENABLED=1 CC=$CC CXX=$CXX GOOS=linux GOARCH=mips go build -v "$@"
```

`./go-mips-build.sh -o agent .`

看看`agent`都依赖哪些动态库：

```txt
# readelf -a agent | grep Shared
 0x00000001 (NEEDED)                     Shared library: [libcapi.so]
 0x00000001 (NEEDED)                     Shared library: [libglib-2.0.so.0]
 0x00000001 (NEEDED)                     Shared library: [libgobject-2.0.so.0]
 0x00000001 (NEEDED)                     Shared library: [libgmodule-2.0.so.0]
 0x00000001 (NEEDED)                     Shared library: [libgio-2.0.so.0]
 0x00000001 (NEEDED)                     Shared library: [libffi.so.6]
 0x00000001 (NEEDED)                     Shared library: [libpthread.so.0]
 0x00000001 (NEEDED)                     Shared library: [libz.so.1]
 0x00000001 (NEEDED)                     Shared library: [libc.so.0]
```

> 提示：使用`ldd`是不可能的了，交叉编译之后，架构都不同，ldd看不出东西来。


![](https://media-1256569450.cos.ap-chengdu.myqcloud.com/blog/15288047680154.jpg)



