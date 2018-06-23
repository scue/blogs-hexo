---
layout: post
title: "解决Android Studio Gobind错误Failed to transform file 'hello.aar' to..."
description: ""
category: 技术
tags: [android, go, gobind]
---

错误表象

```txt
Unable to resolve dependency for ':app@debug/compileClasspath': Failed to transform file 'hello.aar' to match attributes {artifactType=android-exploded-aar} using transform ExtractAarTransform

Unable to resolve dependency for ':app@debugAndroidTest/compileClasspath': Failed to transform file 'hello.aar' to match attributes {artifactType=android-exploded-aar} using transform ExtractAarTransform

Unable to resolve dependency for ':app@debugUnitTest/compileClasspath': Failed to transform file 'hello.aar' to match attributes {artifactType=android-exploded-aar} using transform ExtractAarTransform

Unable to resolve dependency for ':app@release/compileClasspath': Failed to transform file 'hello.aar' to match attributes {artifactType=android-exploded-aar} using transform ExtractAarTransform

Unable to resolve dependency for ':app@releaseUnitTest/compileClasspath': Failed to transform file 'hello.aar' to match attributes {artifactType=android-exploded-aar} using transform ExtractAarTransform
```

最新的解决方法：

<!-- more -->

第一步：将Android Studio更新到`3.1.3`
![](https://media-1256569450.cos.ap-chengdu.myqcloud.com/blog/15297143567652.jpg)


第二步：将工程进行`Clean`
![](https://media-1256569450.cos.ap-chengdu.myqcloud.com/blog/15297143844999.jpg)

第三步：更新`gradle`至4.4版本

![](https://media-1256569450.cos.ap-chengdu.myqcloud.com/blog/15297144058744.jpg)

然后，你可以看到如下效果：

![](https://media-1256569450.cos.ap-chengdu.myqcloud.com/blog/15297144780890.jpg)

以后每次编译都会自动进行gobind了，不像之前必须得手动运行`gomobile bind -a -v -target=android .`, Enjoy ;-) !

