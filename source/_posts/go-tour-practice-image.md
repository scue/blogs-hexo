---
layout: post
date: 2018-02-28 00:00:00
title: "GO指南 练习：图像"
description: ""
category: 技术
tags: 
  - go
---


定义你自己的 Image 类型，实现[必要的方法](https://go-zh.org/pkg/image/#Image)并调用 `pic.ShowImage` 。

Bounds 应当返回一个 `image.Rectangle` ，例如 `image.Rect(0, 0, w, h)` 。

`ColorModel` 应当返回 `color.RGBAModel` 。

At 应当返回一个颜色。上一个图片生成器的值 v 对应于此次的 `color.RGBA{v, v, 255, 255}` 。

```go
package main

import (
	"image"
	"image/color"

	"golang.org/x/tour/pic"
)

type Image struct{}

// ColorModel 应当返回 color.RGBAModel
func (img Image) ColorModel() color.Model {
	return color.RGBAModel
}

// Bounds 应当返回一个 image.Rectangle ，例如 `image.Rect(0, 0, w, h)` 。
func (img Image) Bounds() image.Rectangle {
	return image.Rect(0, 0, 255, 255)
}

// 图片生成器的值 v 对应于此次的 `color.RGBA{v, v, 255, 255}`
func (img Image) At(x, y int) color.Color {
	v := uint8(x % (y + 1))
	return color.RGBA{v, v, 255, 255}
}

func main() {
	m := Image{}
	pic.ShowImage(m)
}

```

效果图：

![](/assets/images/15197989783244.jpg)

链接：[https://tour.go-zh.org/methods/25](https://tour.go-zh.org/methods/25)


