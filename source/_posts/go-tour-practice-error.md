---
layout: post
date: 2018-02-28 00:00:00
title: "GO指南 练习：错误"
description: ""
category: 技术
tags: 
  - go
---


练习：错误
从之前的练习中（牛顿平方根方法）复制 Sqrt 函数，修改它使其返回 error 值。

`Sqrt` 接受到一个负数时，应当返回一个非 nil 的错误值。复数同样也不被支持。

创建一个新的类型

```go
type ErrNegativeSqrt float64
```

并为其实现

```go
func (e ErrNegativeSqrt) Error() string
```

方法使其拥有 error 值，通过 `ErrNegativeSqrt(-2).Error()` 调用该方法应返回 `"cannot Sqrt negative number: -2"` 。

*注意：* 在 Error 方法内调用 `fmt.Sprint(e)` 会让程序陷入死循环。可以通过先转换 e 来避免这个问题：`fmt.Sprint(float64(e))` 。这是为什么呢？

修改 Sqrt 函数，使其接受一个负数时，返回 `ErrNegativeSqrt` 值。


```go
package main

import (
    "fmt"
    "math"
)

type ErrNegativeSqrt float64

func (e ErrNegativeSqrt) Error() string {
    return fmt.Sprintf("cannot Sqrt negative number: %v", float64(e))
}

func Sqrt(x float64) (float64, error) {
    // 当它是一个负数时，返回一个错误
    if x < 0 {
        return 0.0, ErrNegativeSqrt(x)
    }
    z := x / 2
    for i, d, t := 0, 0.0, 0.0; i < 10; i++ {
        t, z = z, z-(z*z-x)/(2*z)
        d = math.Abs(z - t)
        if d < 1e-12 {
            break
        }
    }
    return z, nil
}

func main() {
    fmt.Println(Sqrt(2))
    fmt.Println(Sqrt(-2))
}
```

答：`fmt.Sprint(e)`会优先调用`Error()`方法，即本身，这会导致死循环。

> `ErrNegativeSqrt(x)`：其中`ErrNegativeSqrt`是一个类型，相当于`int`和`float`之类的，而`ErrNegativeSqrt(x)`就表示强制把`x`从`float64`转换为`ErrNegativeSqrt`类型。

链接：https://tour.go-zh.org/methods/20

