---
layout: post
title: "gomobile遇到的问题合集"
description: ""
category: 技术
tags: [GO, gomobile]
---

GO语言支持移动开发，可以直接将已有GO代码编译成一个`libgojni.so`的形式，然后通过Java层代码去调用它，就可以让原本是二进制的程序，变成了一个apk形式去运行。

gomobile目前尚处于实验室阶段，但凡这种实验性的，遇到点问题都是很正常的，比如本文章所述的内容，可能你将来也会遇到。

<!-- more -->

# 问题一：`seq.h: No such file or directory`

这个问题困扰了蛮久，原来是版本号不一致导致的:
```
~/go/src $ gomobile bind -target=android golang.org/x/mobile/example/bind/hello
gomobile: open /Users/scue/go/src/golang.org/x/mobile/bind/java/seq.h: no such file or directory

~/go/src $ ls /Users/scue/go/src/golang.org/x/mobile/bind/java/seq.h
ls: /Users/scue/go/src/golang.org/x/mobile/bind/java/seq.h: No such file or directory
```
解决方式：（升级最新的gomobile和安装二进制）
```
gopm get -g -v -d golang.org/x/mobile
gopm bin -v -d ~/go/bin golang.org/x/mobile/cmd/gomobile
export ANDROID_HOME=/Users/scue/Library/Android/sdk
gomobile init -ndk /Users/scue/source/Android/android-ndk-r16b
gomobile bind -v -target=android golang.org/x/mobile/example/bind/hello
```

# 问题二：Android Studio的Gradle Sync失败

遇到问题，Android Studio的Gradle Sync失败：

```
Unable to resolve dependency for ':app@debug/compileClasspath': Failed to transform file 'hello.aar' to match attributes {artifactType=android-exploded-aar} using transform ExtractAarTransform
```

![](https://ws1.sinaimg.cn/large/6e22ca27gy1fry8nwjxrmj20u006ojsu)

解决方式，按照Github上边的一些提示进行修改：https://github.com/golang/go/issues/23307

若不行，请更新一下你的Android Studio，我这边使用的是Android Studio 3.1.2（目前是最新版本）

> 提示：使用这样子的方式去修改，以后工程代码都需要到相应的工程那里，手动` gomobile bind -v -target=android .`运行一下。



