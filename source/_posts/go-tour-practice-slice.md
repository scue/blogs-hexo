---
layout: post
date: 2018-02-28 00:00:00
title: "GO指南 练习：切片"
description: "描述"
category: 技术
tags: 
  - go
---


实现 `Pic` 。它应当返回一个长度为 `dy` 的切片，其中每个元素是一个长度为 `dx` ，元素类型为 `uint8` 的切片。当你运行此程序时，它会将每个整数解释为灰度值（好吧，其实是蓝度值）并显示它所对应的图像。

图像的选择由你来定。几个有趣的函数包括 `(x+y)/2` 、`x*y` 、 `x^y` 、 `x*log(y)` 和 `x%(y+1)` 。

（提示：需要使用循环来分配 `[][]uint8` 中的每个 `[]uint8` ； 请使用 `uint8(intValue)` 在类型之间转换；你可能会用到 `math` 包中的函数。）



```go
package main

import (
	//"math"
	"golang.org/x/tour/pic"
)

func Pic(dx, dy int) [][]uint8 {
	a := make([][]uint8, dy)
	for i := range a {
		// 很奇怪的地方就是二维数组还需要再make一次
		a[i] = make([]uint8, dx)
		for j := range a[i] {
			//	a[i][j] = uint8(float64(i) * math.Log(float64(j)))
			//	a[i][j] = uint8(i ^ j)
			//	a[i][j] = uint8(i * j)
			a[i][j] = uint8(i % (j + 1))
		}
	}
	return a
}

func main() {
	pic.Show(Pic)
}

```

![效果图](/assets/images/15197857698082.jpg)

链接：https://tour.go-zh.org/moretypes/18

<!-- 备注：巧用range来取得index和value -->




