---
layout: post
title: "如何解决GO语言中的Android DNS解析异常、时区不正确的问题"
description: ""
category: 技术
tags: [GO, Android, DNS, TimeZone]
---

其实这两个问题都是因为Android不是标准的Linux系统环境导致的，如果你给Android操作系统添加上了`/etc/resolv.conf`就能解决DNS问题，时区的问题不容易通过添加文件解决（毕竟要添加的文件还是蛮多的）。

但其实我们作为一个软件开发者，能让自己少一点依赖就少一点依赖，就按Android的开发方式去搞就好了。

<!-- more -->

关于这两个问题，其实在Github上也有人遇到过，但还是有必要将我自己的解决方案给展示一下，做一下记录。

# 关于DNS解析异常的问题

Android的DNS解析是依赖于netd来处理的，我们可以通过`getprop`来拿到DNS1和DNS2的解析服务器配置信息。

但这些我们其实不用Care，解决这一个问题的方式最好的是使用配置和使用Android的交叉工具链去编译程序即可。

## 下载NDK

从官网下载：https://developer.android.com/ndk/downloads/index.html?hl=zh-cn

![](https://ws1.sinaimg.cn/large/6e22ca27gy1fr2175gvcrj21f00hqtef)

我这里是Mac环境，就是下载了mac的版本。

https://dl.google.com/android/repository/android-ndk-r16b-darwin-x86_64.zip


## 解压到一个位置
例如：`/Users/scue/source/Android/android-ndk-r16b`

## 生成独立工具链

由于我的开发环境是Android SDK版本是25（7.1.2），因些我导出的是`--platform=android-24`。

`build/tools/make-standalone-toolchain.sh --verbose --arch=arm64 --platform=android-24 --install-dir=/Users/scue/source/Android/arm64-android-toolchain-target24`

## 写一个编译脚本

由于交叉编译环境配置的变量较多，建议写一个脚本来编译关于Android的二进制程序。

比如我就写在了`~/bin/go-build-android.sh`里边，内容如下：

```sh
#!/bin/sh

set -ex

export ANDROID_NDK=/Users/scue/source/Android/android-ndk-r16b
export TOOLCHAINS=/Users/scue/source/Android/arm64-android-toolchain-target24
export CC=${TOOLCHAINS}/bin/aarch64-linux-android-clang
export CXX=${TOOLCHAINS}/bin/aarch64-linux-android-clang++
export GOMOBILE=$HOME/go/pkg/gomobile
export GOOS="android"
export GOARCH="arm64"
export CGO_CFLAGS="-target aarch64-none-linux-android --sysroot ${TOOLCHAINS}/sysroot"
export CGO_CPPFLAGS="-target aarch64-none-linux-android --sysroot ${TOOLCHAINS}/sysroot"
export CGO_LDFLAGS="-target aarch64-none-linux-android --sysroot ${TOOLCHAINS}/sysroot"
# export GOARM=7
export CGO_ENABLED=1
go build -a -pkgdir="$GOMOBILE/pkg_android_arm64" -ldflags '-extldflags "-pie -fPIE -fPIC"' "$@"
```

接下来我就可以直接使用这个脚本编译GO源码了。

## 编译测试

写一个简单的程序

```go
package main

import (
    "fmt"
    "log"
    "net"
    "time"
)

func main() {
    fmt.Println("Hello, GO!")
    log.Println(net.LookupIP("tw07a102.sandai.net"))
    log.Println(time.Now())
}
```

编译一下：
`~/bin/go-build-android.sh -o test test.go`

然后我们将文件通过`adb push test /data/local/tmp/test`推送到Android环境。

运行结果：

```txt
WARNING: linker: /data/local/tmp/test: unsupported flags DT_FLAGS_1=0x8000000
Hello, GO!
2018/05/04 03:25:29 [180.97.84.130] <nil>
2018/05/04 03:25:29 2018-05-04 03:25:29.165968761 +0000 UTC m=+0.004954710
```

其实绕这么大的一个弯子来解决这个问题，也是值得的，毕竟以后也不晓得使用其他编译方式会出什么乱子。这样子配置好了编译脚本，只要涉及到要移植到Android上工作的，通通使用它编译，就不会有太多的麻烦了。

# 关于时区不正确的问题


关于时区不对的问题，可以通过在`init`函数上，通过调用`getprop persist.sys.timezone`来拿到时区信息，比如我这里拿到的是`Asia/Shanghai`，也就是和我们日常生活中的北京时间是一致的。

```go
func FixTimezone() {
	out, err := exec.Command("/system/bin/getprop", "persist.sys.timezone").Output()
	if err != nil {
		return
	}
	z, err := time.LoadLocation(strings.TrimSpace(string(out)))
	if err != nil {
		return
	}
	time.Local = z
}
```

然后可以在`package main`这个模块的init函数中调用一下这个`FixTimezone`即可。

