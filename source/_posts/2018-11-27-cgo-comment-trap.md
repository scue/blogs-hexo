---
layout: post
title: "CGO开发中必须注意的一个注释陷阱"
description: ""
category: 技术
tags: [cgo]
---

CGO开发中，如果有注释使用不当，你会遇到一些奇奇怪怪的问题。比如下方的一个使用示例就是一个典型：

<!-- more -->

```go
package main

/**
#include <stdio.h>
#include <sys/utsname.h>
struct utsname name;
void uname_main() {
	uname(&name);
	printf("uname sysname: %s\n", name.sysname);
	printf("uname nodename:%s\n", name.nodename);
	printf("uname release: %s\n", name.release);
	printf("uname version: %s\n", name.version);
	printf("uname machine: %s\n", name.machine);
}
*/
import "C"
import (
	"sync"
	"unsafe"

	log "github.com/Sirupsen/logrus"
)

var (
	unameOnce sync.Once
)

type AFormatter struct{}

func (f *AFormatter) Format(entry *log.Entry) ([]byte, error) {
	return []byte(entry.Message), nil
}

func init() {
	log.SetFormatter(&AFormatter{})
}

//noinspection GoAddressOperators
func main() {
	unameOnce.Do(func() {
		C.uname_main()
	})
	log.Printf("C.sysname: %v", C.GoString((*C.char)(unsafe.Pointer(&C.name.sysname))))
	log.Printf("C.nodename: %v", C.GoString((*C.char)(unsafe.Pointer(&C.name.nodename))))
	log.Printf("C.release: %v", C.GoString((*C.char)(unsafe.Pointer(&C.name.release))))
	log.Printf("C.version: %v", C.GoString((*C.char)(unsafe.Pointer(&C.name.version))))
	log.Printf("C.machine: %v", C.GoString((*C.char)(unsafe.Pointer(&C.name.machine))))
}

```

这段代码想使用CGO来调用uname，获取计算机的一些uname信息，我们来编译一下看看输出：

```txt
# command-line-arguments
In file included from onething.net/onecloud/test/main.go:4:
In file included from /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.14.sdk/usr/include/stdio.h:64:
In file included from /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.14.sdk/usr/include/_stdio.h:71:
In file included from /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.14.sdk/usr/include/_types.h:27:
In file included from /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.14.sdk/usr/include/sys/_types.h:33:
In file included from /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.14.sdk/usr/include/machine/_types.h:32:
/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.14.sdk/usr/include/i386/_types.h:37:1: error: expected identifier or '('
typedef __signed char           __int8_t;
^
1 error generated.

Compilation finished with exit code 2
```

这段代码，如果使用C语言来直接编译和输出是完全没有问题的，输出如下：
```txt
uname sysname: Darwin
uname nodename:XXX-MacBook-Pro.local
uname release: 18.0.0
uname version: Darwin Kernel Version 18.0.0: Wed Aug 22 20:13:40 PDT 2018; root:xnu-4903.201.2~1/RELEASE_X86_64
uname machine: x86_64
```

解决这个问题很简单，就是把多余的星号去掉！！

![](https://media-1256569450.cos.ap-chengdu.myqcloud.com/blog/15432879151906.jpg)

正确的输出：

![](https://media-1256569450.cos.ap-chengdu.myqcloud.com/blog/15432880444342.jpg)

是不是很惊喜，是不是很意外？！查了我好久的时间... <哭泣>.png