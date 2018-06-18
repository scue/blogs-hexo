---
layout: post
title: "你一直在寻找的GO语言条件编译"
description: ""
category: 技术
tags: [GO, build]
---

- 你还在寻找类似于像C语言的`#if defined`条件编译吗？

- 你的Go程序运行于<font color=#f58220>多个平台上（如Linux和Windows）</font>，相同功能有着不同的实现，你知道怎么处理吗？

- 你的Go程序运行于<font color=#d93a49>多代产品上（如一代盒子，二代盒子）</font>，相同功能有着不同的实现，你知道怎么解决吗？

阅读本文章，对于解决以上的疑惑，一定有所帮助。

<!-- more -->


# 一、误入歧途

GO语言没有类似于C语言的`#if defined`功能，如果不了解GO语言的条件编译功能，很可能会误入歧途，做出如下决策：

- 将源码依据<font color=#ef4136>不同的平台、不同的产品</font>创建不同的分支（git branch）
- 区分不同平台时，大量地使用`if "android" == runtime.GOOS`这样子的判断语句

这会导致什么问题？

- 两代产品之间，相同的需求，需要两个人去维护，变更需求的时候反反复复地合并相同的代码
- 长期维护下去需要消耗很大的精力，并且很容易出现差错


# 二、迷途知返

GO语言条件编译官网：https://golang.org/pkg/go/build/#hdr-Build_Constraints

## 不同平台的条件编译

偶然的一次，发现GO语言的一些代码长成这样子：

![](https://media-1256569450.cos.ap-chengdu.myqcloud.com/blog/15293001813846.jpg)

对应的`sys_windows.go`内容如下：

![](https://media-1256569450.cos.ap-chengdu.myqcloud.com/blog/15293002490870.jpg)

对应的`sys_linux.go`内容如下：

![](https://media-1256569450.cos.ap-chengdu.myqcloud.com/blog/15293002878153.jpg)

这很明显是两个平台之间，相同的函数有着不同的实现！

> 印象中，GO语言是不能存在相同的函数的，即不支持函数重载。后来阅读了官网的资料之后，才知道，其实编译过程中，若是只编译linux的时候，`sys_windows.go`是不会被编译的，Go语言只抽取了它所需要的文件`sys_linux.go`进行编译，这点真高效。

有了这个，我们可以做很多文章，比如像这样子：

![](https://media-1256569450.cos.ap-chengdu.myqcloud.com/blog/15293005927260.jpg)

- hello_darwin: MacOS
- hello_windows: Windows
- hello_linux: Linux
- hello_android: linux, android

MacOS的运行结果：

```txt
$ go run main.go
2018/06/18 13:43:28 Hello, darwin!
```

### Android平台存在的坑点

![](https://media-1256569450.cos.ap-chengdu.myqcloud.com/blog/15293009279820.jpg)

官方网站有提到`Using GOOS=android matches build tags and files as for GOOS=linux in addition to android tags and files.`，当使编译Android平台的时候，会把Linux相关的文件一起加入进行编译，优先编译了`_linux.go`文件，然后再到`_android.go`文件，解决方法看下一小节的` 两代产品间的条件编译`。

## 两代产品间的条件编译

我们先去[GO语言官网Build Constraints](https://golang.org/pkg/go/build/#hdr-Build_Constraints)查看一下:

![](https://media-1256569450.cos.ap-chengdu.myqcloud.com/blog/15292999674365.jpg)

GO语言的条件编译只需要在文件起始的位置添加上`// +build的注释`即可。

GO语言除了可以支持使用文件名来区分不同的平台以外，还支持使用`-tags`来区分不同的编译，比如使用`-tags boxone`来表示这是编译第一代的产品。

第一代产品的go文件：

![](https://media-1256569450.cos.ap-chengdu.myqcloud.com/blog/15293014578933.jpg)

第二代产品的go文件：

![](https://media-1256569450.cos.ap-chengdu.myqcloud.com/blog/15293014811510.jpg)

运行效果：

![](https://media-1256569450.cos.ap-chengdu.myqcloud.com/blog/15293015894651.jpg)

官网说得不是很清楚，其实`// +build boxone`的作用是，仅在`tags`包含了`boxone`的时候，这个文件才会被被编译。

### 怎么解决Android编译有冲突的问题？

自从有了tags之后，解决这个问题大家应该都能想得到了，**最好优先使用tags来区分不同的产品**。

如果实在需要`_linux.go`和`_android.go`作出明确的区分，需要在`_linux.go`，使用`// +build !android`来解决问题。

![](https://media-1256569450.cos.ap-chengdu.myqcloud.com/blog/15293023786695.jpg)


### Intellij Idea的配置

Intellij Idea支持设定编译的tags，这样子可以在写代码的时候进行一些错误检查。

![](https://media-1256569450.cos.ap-chengdu.myqcloud.com/blog/15293013313590.jpg)




