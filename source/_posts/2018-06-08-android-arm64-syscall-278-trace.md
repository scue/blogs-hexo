---
layout: post
title: "追踪Android内核系统调用Syscall 278堆栈错误心路历程"
description: ""
category: 技术
tags: [Android, Kernel, Syscall]
---

# 错误表象

最近使用Golang开发了一个内部的Android APK，当提交到测试的时候，遇到非必现的出现以下错误：

![](https://media-1256569450.cos.ap-chengdu.myqcloud.com/blog/15284272908771.jpg)

<!-- more -->

完整日志：http://paste.ubuntu.org.cn/4339707

# 开始定位

- 这个又不像Oops日志，整体内容也很难看
- 只有一个PC和LR的地址，还没有Backstrace
- 那只好对vmlinux使用gdb或addr2line进行操刀了
- 这里使用addr2line定位一下具体出错的函数：
- 命令：`aarch64-linux-android-addr2line -fe $OUT/obj/KERNEL_OBJ/vmlinux 72369a3d58`

  ![](https://media-1256569450.cos.ap-chengdu.myqcloud.com/blog/15284273540379.jpg)
  
- 这都是什么鬼？？定位不出来？

# 回到起点

- 现在唯一的线索就是知道它调用了syscall 278
- 我们去看一下内核的定义：`common/arch/arm64/include/asm/unistd32.h`
  ![](https://media-1256569450.cos.ap-chengdu.myqcloud.com/blog/15284273784880.jpg)
- `mq_notify`是什么鬼？我好像在程序中没有使用过你呀？

# 绕个弯子
- 发现在linux 64位的环境下使用，没有问题的
- 现在来看一下NDK工具链的系统调用表：`${TOOLCHAINS}/sysroot/usr/include/asm-generic/unistd.h`
  
  ![](https://media-1256569450.cos.ap-chengdu.myqcloud.com/blog/15284274749018.jpg)
  
- 可以清楚的看到Android NDK里边`278`对应的是`getrandom`系统调用

# 我们写个Demo

- 编译：`aarch64-linux-android-gcc -pie -fPIE -fPIC syscall-test.c -o syscall-test`
- 测试一下，然后看看内核堆栈：

  ![](https://media-1256569450.cos.ap-chengdu.myqcloud.com/blog/15284275769956.jpg)

- 没错就这样子，又出现堆栈错误啦~
# 结论？
- 我们再回去看看内核的系统调用：`common/arch/arm64/include/asm/unistd32.h`
- 发现`syscall 278`对应并不是`getrandom`
- 并且我们内核似乎也没有对`getrandom`有注册系统调用
- 所以，问题就在这里，NDK有定义的系统调用，在内核里边并没有！

# getrandom是什么？
- 它是一个获取随机数的系统调用
- 来一个Linux版本的示例：

  ![](https://media-1256569450.cos.ap-chengdu.myqcloud.com/blog/15284276498385.jpg)

- 这个通常在加密的时候会使用它来获取随机的数值，毕竟我们写的代码加密的东西还蛮多的

# 解决方式
- 让固件团队去修改内核这是一种方式
- 同时我这边也做了一些规避的措施，改的是GO底层库一些相关东西，比较Low的做法，就不展示了~






