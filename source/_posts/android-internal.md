---
layout: post
date: 2015-05-11 00:00:00
title: "课程3: Android系统知识(一)"
description: ""
category: 课程
tags:
  - 课程
---

# Android系统知识

这一部分主要针对Androi系统整个框架，列出几个重要的Android系统关系图，进行讲解一下；

因为Android基于Linux内核，所以先从Linux基础开始讲解，然后再逐渐引入Android系统框架；

### Linux Kernel Map

Linux Kernel Map这一张图主要是针对开发人员提供的，有兴趣的童鞋可以大致了解一下；

![Linux-Kernel-Map](/assets/Courses/Courses03_android_internal_02.LKM3_1024.png)

[源链接](http://www.makelinux.net/kernel_map/)

若对Android Kernel与Linux Kernel的差异感兴趣，可以参考这里 [ANDROID LINUX KERNEL ADDITIONS](http://www.lindusembedded.com/blog/2010/12/07/android-linux-kernel-additions/)

简单列举如下：

- `binder`:                             进程通信驱动
- `ashmem`:                             匿名共享内存驱动
- `pmem`:                               process memory allocator
- `logger`:                             打印日志驱动
- `wake locks`:                         一但有用户层、或Kernel拿到此锁就阻止手机进入低功耗状态
- `early suspend`:                      Linux电源管理的扩展，LCD、重力感应、传感器依赖于它才能在睡眠时工作；
- `oom handling`:                       android out of memory killer, 内存管理（以前Android手机内存很小，这个很有必要）
- `android alarm`:                      闹钟服务，允许用户空间控制闹钟
- `android paranoid network security`:  通过uid控制网络访问权限
- `android timed output/gpio`:          暴露定时gpio模块到用户空间，供震动模块调用
- `android ram console`:                允许保存一部分printk信息到一小段内存上`/proc/last_kmsg`，方便调试
- `other android differences`:          yaffs2, bluetooth, scheduleer, adb

### Map of GNU/Linux OS Internals

这是一张GNU/Linux系统各层调用关系图，利用它，我们可以进一步完善SSLVPN、VMP和VDC等案例的设计；

![Linux-OS-Internel](/assets/Courses/Courses03_android_internal_01.GNU_Linux_OS_internals.png)

[源链接](http://www.makelinux.net/system/GNU_Linux_OS_internals.png)

### Android Internal

这一部分是Android系统各层间调用关系图

在我们遇到一些问题不容易排解的时候，看此图，上下左右关系看一看，就能找到答案了

![Android-Internal](/assets/Courses/Courses03_android_internal_03.Android_internals.png)

[源链接](http://www.makelinux.net/android/internals/Android_internals_1024.png)

### Android System Architecture

这一部分是Android系统结构图，来自于Google Android官网

![Android System Architecture](/assets/Courses/Courses03_android_internal_05.Android-system-architecture.jpg)

[源链接](https://developer.android.com/images/system-architecture.jpg)

### Android API Classes

![Android API Classes](/assets/Courses/Courses03_android_internal_04.Android_API_Classes.png)

[源链接](http://www.makelinux.net/android/classes/zoom)

### 更多

更多可以查看此[链接](http://www.makelinux.net/resources)
