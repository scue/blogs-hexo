---
layout: post
title: "CGO: C语言调用GO语言的函数"
description: ""
category: 技术
tags: [cgo, c, go]
---

网上有很多例子告诉你怎么从Go语言调用C语言的函数，但鲜少文章有告诉你，其实CGO开发中，还可以从C语言函数中，调用Go语言写的函数，而你只需要简单的加一个`// export`注释就可以了。

<!-- more -->

以下是一个简单的例子：

```go
package main
/*
#include <stdlib.h>
void print_str(const char *s);
*/
import "C"
import "unsafe"
func Print(s string) {
    cs := C.CString(s)
    C.print_str(cs)
    C.free(unsafe.Pointer(cs))
}
func main() {
    str1 := "hi how are you\n"
    Print(str1)
}
//export HelloByGo
func HelloByGo(name string) *C.char {
    return C.CString("greeting " + name)
}
```
> 提示：需要添加 `//export HelloByGo`来声明此函数是暴露给C语言使用的

```c
#include "stdio.h"
#include "stdlib.h"
#include "_cgo_export.h"
void print_str(const char *s)
{
    printf("%s\n", s?s:"nil");
    // C语言调用 Go语言函数
    GoString name;
    name.p = "123";
    name.n = 3;
    char *greeting = HelloByGo(name);
    printf("call go: %s\n", greeting);
    free(greeting);
}
```

运行结果：

```txt
root@ec6da906bfea:/work# go run .
hi how are you
call go: greeting 123 // C调用go语言编写的函数
```

> 提示：需要引入`#include "_cgo_export.h"`来表示使用cgo生成的函数

GO语言常见类型在C的实现表现:

```c
typedef struct { char *p; int n; } GoString;
typedef struct { void *data; int len; int cap; } GoSlice;
typedef struct { void *t; void *v; } GoInterface;
```

参考资料：

1. [cgo, GoSlice/GoString from C](https://groups.google.com/forum/#!topic/golang-nuts/bBylU8kE5uk)
2. [C_references_to_Go, Go functions can be exported for use by C code](https://golang.org/cmd/cgo/#hdr-C_references_to_Go)