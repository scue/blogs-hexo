---
layout: post
title: "CGO类型及常见类型转换"
description: ""
category: 技术
tags: [cgo]
---

关于Cgo的相关类型描述，官网上有这么一段话：

The standard C numeric types are available under the names C.char, C.schar (signed char), C.uchar (unsigned char), C.short, C.ushort (unsigned short), C.int, C.uint (unsigned int), C.long, C.ulong (unsigned long), C.longlong (long long), C.ulonglong (unsigned long long), C.float, C.double, C.complexfloat (complex float), and C.complexdouble (complex double). The C type void* is represented by Go's unsafe.Pointer. The C types __int128_t and __uint128_t are represented by [16]byte.

<!-- more -->

| Name | Description |
| --- | --- |
| C.char | signed char |
| C.uchar | unsigned char |
| C.short | short int |
| C.ushort | unsigned short int |
| C.int | int |
| C.uint | unsigned int |
| C.long | long |
| C.ulong | unsigned long |
| C.float | float |
| C.double | unsigned float |
| C.complexfloat | complex float |
| C.complexdouble | complex double |
| unsafe.Pointer | void* |
| [16]byte | __int128_t, __uint128_t |

常见转换如下：

字节转换指针：
`[]byte → char *`: `pData = (*C.char)(unsafe.Pointer(&data[0]))`
整数转换指针：
`int →int *`: `pFlag := (*C.int)(unsafe.Pointer(&flag))`

> PS：很多时候，我更倾向于上边这样子的使用方法，这样子在C语言中就不必malloc和free了，而把内存管理交给了GO语言这边来处理，用完就自动释放了。

当然，你也可以使用官方的，如字符串转换：
```go
cs := C.CString("Hello from stdio")
C.myprint(cs)
C.free(unsafe.Pointer(cs))
```

参考链接：[Command cgo](https://golang.org/cmd/cgo/)