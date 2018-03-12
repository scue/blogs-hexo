---
layout: post
date: 2018-02-28 00:00:00
title: "GO指南 练习：rot13Reader"
description: "描述"
category: 技术
tags: 
  - go
---



有种常见的模式是一个 `io.Reader` 包装另一个 `io.Reader` ，然后通过某种方式修改其数据流。

例如，`gzip.NewReader` 函数接受一个 `io.Reader` （已压缩的数据流）并返回一个同样实现了 `io.Reader` 的 `*gzip.Reader` （解压后的数据流）。

编写一个实现了 `io.Reader` 并从==另一个 `io.Reader` 中读取数据的 `rot13Reader`== ， 通过应用 `rot13` 代换密码对数据流进行修改。

`rot13Reader` 类型已经提供。实现 `Read` 方法以满足 `io.Reader` 。


```go
package main

import (
	"io"
	"os"
	"strings"
)

type rot13Reader struct {
	r io.Reader
}

// 实现加密函数
func rot13(b byte) byte {
	var a byte
	switch {
	case b >= 'a' && b <= 'z':
		a = 'a'
	case b >= 'A' && b <= 'Z':
		a = 'A'
	default:
		return b
	}
	return (b-a+13)%26 + a
}

// Read(p []byte) (n int, err error)
func (r rot13Reader) Read(b []byte) (n int, err error) {
	// 先从源数据上读取内容
	n, err = r.r.Read(b)
	for i := 0; i < n; i++ {
		b[i] = rot13(b[i])
	}
	return
}

func main() {
	s := strings.NewReader("Lbh penpxrq gur pbqr!")
	r := rot13Reader{s}
	io.Copy(os.Stdout, &r)
}

```

注意：首先需要从`rot13Reader.r`这个`io.Reader`上读取得到数据，再进一步使用`rot13`函数进行加密。

输出结果：


```txt
You cracked the code!
Program exited.
```


