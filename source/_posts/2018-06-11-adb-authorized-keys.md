---
layout: post
title: "Android授权ADB USB调试的技巧"
description: ""
category: 技术
tags: [Android, ADB]
---

假如你只有串口调试，而串口输出又狠狂暴。

此时此刻，你多么想拥有ADB调试。

那你知道如何不需要给板子连接`显示器`、`鼠标`和`键盘`这三件的情况下，对某台电脑USB调试进行授权吗？

<!-- more -->

# 方法一

通过 `input`命令，它的使用帮助如下：

```txt
Usage: input [<source>] <command> [<arg>...]

The sources are:
      keyboard
      mouse
      joystick
      touchnavigation
      touchpad
      trackball
      dpad
      stylus
      gamepad
      touchscreen

The commands and default sources are:
      text <string> (Default: touchscreen)
      keyevent [--longpress] <key code number or name> ... (Default: keyboard)
      tap <x> <y> (Default: touchscreen)
      swipe <x1> <y1> <x2> <y2> [duration(ms)] (Default: touchscreen)
      press (Default: trackball)
      roll <dx> <dy> (Default: trackball)
```

然后我们就可以在串口上使用 `input tap <x> <y>` 来模拟对`确定`键进行点击。

这个的缺点很明显，我需要知道这个确定键的坐标...

# 方法二（推荐）

第一步，先`cat ~/.android/adbkey.pub`拿到自己的公钥

![](https://media-1256569450.cos.ap-chengdu.myqcloud.com/blog/15287177849841.jpg)

第二步，将这个公钥`追加`到Android的`/data/misc/adb/adb_keys`文件里边。

我这有一气呵成的命令：

```sh
mkdir -p /data/misc/adb && \
echo 'ADB_KEY_PUB' >> /data/misc/adb/adb_keys && \
chmod 664 /data/misc/adb/adb_keys && \
stop adbd && start adbd
```

`ADB_KEY_PUB`修改成你自己的KEY即可。~


