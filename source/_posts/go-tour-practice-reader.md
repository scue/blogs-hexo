---
layout: post
date: 2018-02-28 00:00:00
title: "GO指南 练习：Reader"
description: ""
category: 技术
tags: 
  - go
---


练习：Reader

实现一个 Reader 类型，它产生一个 ASCII 字符 `'A'` 的无限流。

```go
package main

import "golang.org/x/tour/reader"

type MyReader struct{}

// TODO: Add a Read([]byte) (int, error) method to MyReader.
func (r MyReader) Read(b []byte) (n int, err error) {
	for i := range b {
		b[i] = 'A'
	}
	return len(b), nil
}

func main() {
    reader.Validate(MyReader{})
}
```

链接：https://tour.go-zh.org/methods/22



