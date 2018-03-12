---
layout: post
date: 2015-09-01 00:00:00
title: "Android Studio 使用笔记"
description: ""
category: "技术"
tags: 
  - Studio
---

## 下载链接

官方网站: [developer.android.com/tools/studio](http://developer.android.com/tools/studio/index.html)

由于官网需要特殊方法才能正常访问，所以，我是从 [这里下载](http://android-studio.org/) 的；

## 问题汇总

*   `Error:(2) Error retrieving parent for item: No resource found that matches the given name 'android:TextAppearance.Material.Widget.Button.Inverse'`

    跟踪链接: `https://code.google.com/p/android/issues/detail?id=183122`
    依据[#31](https://code.google.com/p/android/issues/detail?id=183122#c31): 修改`app/build.gradle`

        -compile 'com.android.support:appcompat-v7:23.0.0'
        +compile 'com.android.support:appcompat-v7:22.2.1'
