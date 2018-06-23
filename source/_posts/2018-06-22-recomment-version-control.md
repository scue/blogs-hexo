---
layout: post
title: "推荐的Android App和Rom版本号定义方式"
description: ""
category: 技术
tags: [android, version]
---

推荐使用[语义化版本2.0.0](https://semver.org/lang/zh-CN/)，比如`1.0.0`

对应到Android App的Studio工程配置：

<!-- more -->

```gradle
def versionMajor = 1
def versionMinor = 0
def versionPatch = 3 // patch是10, 20, 30, 40, ...的时候是正式发布版本，其余为测试版本
def versionBuild = 1 // developer自己使用
android {
    compileSdkVersion 27
    defaultConfig {
        applicationId "com.example.app"
        versionCode versionMajor * 10000 + versionMinor * 1000 + versionPatch * 100 + versionBuild
        versionName "${versionMajor}.${versionMinor}.${versionPatch}"
    }
}
```

- 提测版本：`1.0.1-1.0.9`, `1.0.11-1.0.19`等等
- 正式版本：`1.0.0`, `1.0.10`, `1.0.20` ...
- 开发自测：自己去修改`versionBuild`就可以了，范围是`0~99`自己随意定~

Android的则对应到`ro.build.display.id`属性上。


福利：

1. NodeJs服务端如何快捷比较版本：https://www.npmjs.com/package/semver
2. Golang服务端如何快捷比较版本：https://github.com/Masterminds/semver

