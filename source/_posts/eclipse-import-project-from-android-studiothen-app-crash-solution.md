---
layout: post
date: 2015-05-09 00:00:00
title: "Android Studio迁移闪退"
description: ""
category: 技术
tags: 
  - Android
---
很久没有撸Android App开发了～
最近把一个月前通过反编译、二次修改的 [Android SSHD](https://code.csdn.net/myscue/droidsshd/tree/lyh_eclipse)  项目进行简单修改一下；
突然发现迁移项目时，报了一个错误，同时还出现了闪退情况：
    
    04-29 20:20:11.493: W/dalvikvm(23964): threadid=1: thread exiting with uncaught exception (group=0x41b2cc50)
    04-29 20:20:11.543: E/StubController(23964): service = null

感觉像是迁移过程中没完整导致的，解决方法很简单，**就是把java目录重命名为src即可**；

![solution](/assets/images/292045497241929.png)

[历史提交记录](https://code.csdn.net/myscue/droidsshd/commit/70d72506802f4d0034b62889a6c6d28f29a6b217)
