---
layout: post
date: 2018-02-28 00:00:00
title: "GO指南 练习：斐波纳契闭包"
description: ""
category: 技术
tags: 
  - go
---


练习：斐波纳契闭包

让我们用函数做些好玩的事情。

实现一个 `fibonacci` 函数，它返回一个函数（闭包）， 该闭包返回一个斐波纳契数列 `(0, 1, 1, 2, 3, 5, ...)` 。

```go
package main
import "fmt"

func fibonacci() func() int {
   a, b := 0, 1
   return func() (v int) {
      v, a, b = a, b, a+b
      return
   }
}

func main() {
	f := fibonacci()
	for i := 0; i < 10; i++ {
		fmt.Println(f())
	}
}

```


链接：https://tour.go-zh.org/moretypes/26

