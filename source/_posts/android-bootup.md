---
layout: post
date: 2015-05-20 00:00:00
title: "课程3: Android启动过程分析"
description: ""
category: 课程
tags: 
    - 课程
    - Android
---

#   相关技术文章

首先参阅一下别人写得很不错的文章，个人感觉还是有很多可以借鉴的内容

##   1.  Android启动过程深入解析 [原文链接](http://blog.jobbole.com/67931/)

当然，小伙伴们若是英文足够好，可以直接查看：[英文原文](http://kpbird.blogspot.com/2012/11/in-depth-android-boot-sequence-process.html)

![Android_bootup1](/assets/Courses/Courses03_android_boot_squence.png)

我们先来粗略查看一下文章要点

####第一步：启动电源以及系统启动

>当电源按下，引导芯片代码开始从预定义的地方（固化在ROM）开始执行。

####第二步：引导程序

>引导程序是运行的第一个程序，因此它是针对特定的主板与芯片的；

>引导程序是OEM厂商或者运营商加锁和限制的地方;

####第三步：内核

>Android内核与桌面linux内核启动的方式差不多。

####第四步：init进程

>init进程有两个责任，一是挂载目录，比如/sys、/dev、/proc，二是运行init.rc脚本

####第五步：Zygote

>Zygote让Dalvik虚拟机共享代码、低内存占用以及最小的启动时间成为可能。

>Zygote是一个虚拟器进程，正如我们在前一个步骤所说的在系统引导的时候启动。Zygote预加载以及初始化核心库类。

####第六步：系统服务或服务

>系统服务同时使用native以及java编写，系统服务可以认为是一个进程。

>系统服务包含了所有的System Services。

>Zygote创建新的进程去启动系统服务。你可以在ZygoteInit类的”startSystemServer”方法中找到源代码。

####第七步：引导完成

>`ACTION_BOOT_COMPLETED` 启动已完成的广播

备注：在init.rc上还可以使用`on property:dev.bootcomplete=1`来处理你期望开机完成后做爱做的事情

##  2. Android启动分析 [原文链接](http://blog.csdn.net/dlmu2001/article/details/6537304)

由于前一篇文章，在`system_server或services`上讲解不够深入，现在利用这一篇文章来粗略讲解一下

关于这个文章的英文原文: [链接: Android Start Up](http://www.phonesdevelopers.com/1695827/)

接下来这一段，我是依据 [链接: 2.3服务启动过程](http://blog.csdn.net/dlmu2001/article/details/6537304#t8) 来讲解有关 `system_server`和`Services`的内容

####1. `daemons`和`Zygote`

>1) `init.rc`启动类似`adbd`, `debuggerd`, `rild`等核心后台进程，用于监听指定信息

>2) `Zygote`是用于初始化虚拟机的进程，监听请求创建虚拟机实例的socket，算是App的“鼻祖”

![2.1_init.rc_daemons_zygote](/assets/Courses/Courses03_android_bootup_01.gif "Android 启动分析：Daemons和Zygote")

####2. `Services Manager`

在 [init.rc](http://androidxref.com/4.4_r1/xref/system/core/rootdir/init.rc#448) 有这么一段内容

    service servicemanager /system/bin/servicemanager
        class core
        user system
        group system
        critical
        onrestart restart healthd
        onrestart restart zygote
        onrestart restart media
        onrestart restart surfaceflinger
        onrestart restart drm

>init.rc启动Service Manager，注册为Binder服务缺省的context manager，处理服务注册和监听

![2.2_init.rc_servicemanager](/assets/Courses/Courses03_android_bootup_02.gif "Android 启动分析：Services Manager")

####3. `system_server`

system_server是由Zygote分裂出来的一个进程(而非原文所说的由Runtime拉起)

`争议`：这里原文可能是针对较低版本的，经过分析源码，正确的应该是 [链接: system_server启动流程](http://images.cnitblog.com/blog/563439/201309/25154945-40c8b0ec63774e02af485973a4c05c45.png)

![system_server_start_up_seq_bad](/assets/Courses/Courses03_android_bootup_04.gif)

####4. `System server` init1和init2

在`system_server`起来之后，它调用了`init1`和`init2`

>init1: 拉起了`SurfaceFlinger`和`AudioFlinger`等（与硬件层交互的服务，主要是C/C++代码）

![init1](/assets/Courses/Courses03_android_bootup_05.gif "init1")

####5. 原生`server sevices`向`service manager`注册为IPC服务目标

比如AudioFlinger.cpp的instantiate函数

>init2: 拉起了`Activity Manager`和`Window Manager`等（与应用层、框架层交互的服务，主要是Java代码）

![init2](/assets/Courses/Courses03_android_bootup_06.gif "init2")

####6. `Android managed services`

SystemServer.java的run函数

![Managed Services](/assets/Courses/Courses03_android_bootup_07.gif)

####7. `Android managed services`向`service manager`注册

![Managed Services](/assets/Courses/Courses03_android_bootup_08.gif)

####8. `system_server`加载所有的服务后，系统初始化完成

![system_server](/assets/Courses/Courses03_android_bootup_09.gif)


####9. 桌面程序

`ActivityManagerService.java`的`startHomeActivityLocked`

![HomeActivity](/assets/Courses/Courses03_android_bootup_10.gif)

####10.其他App在自己进程中启动

![OtherActivity](/assets/Courses/Courses03_android_bootup_11.gif)

##  3. 图解Android - Zygote, System Server 启动分析 [原文链接](http://www.cnblogs.com/samchen2009/p/3294713.html)

这绝对是一篇好文，写得很精彩

相应的，这个博主的Github相关链接: [android_url](https://github.com/samchen2009/android_uml)

我对这里的`system_server`启动流程比较感兴趣，这里只贴出`system_server`相关的启动流程图

#### `system_server`启动流程图

![SystemServerUml](/assets/Courses/Courses03_android_bootup_systemServer.png "Android 启用分析: SystemServer启动流程图")
