---
layout: post
date: 2018-02-28 00:00:00
title: "GO指南 练习：循环与函数"
description: "牛顿法实现平方根函数"
category: 技术
tags: 
  - go
---

计算机通常使用循环来计算 x 的平方根。从某个猜测的值 z 开始，我们可以根据 z² 与 x 的近似度来调整 z，产生一个更好的猜测：

```
z -= (z*z - x) / (2*z)
```

修改循环条件，使得当值停止改变（或改变非常小）的时候退出循环。

具体我的实现：

```go
package main

import (
    "fmt"
    "math"
)

func Sqrt(x float64) float64 {
    z := x / 2
    // 当循环至10次、或两次值差异只有1e-12时停止循环
    for i, d, t := 0, 0.0, 0.0; i < 10; i++ {
        t, z = z, z-(z*z-x)/(2*z)
        d = math.Abs(z - t)
        if d < 1e-12 {
            break
        }
    }
    return z
}

func main() {
    fmt.Println(Sqrt(2))
    fmt.Println(Sqrt(3))
    fmt.Println(Sqrt(5))
    fmt.Println(Sqrt(6))
    fmt.Println(Sqrt(7))
}

```

练习链接：https://tour.go-zh.org/flowcontrol/8


