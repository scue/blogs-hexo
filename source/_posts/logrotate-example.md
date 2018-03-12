---
layout: post
date: 2018-02-11 00:00:00
title: "使用lorotate压缩服务器日志"
description: "logrotate"
category: 技术
tags: 
  - linux
---

马上春节了，发现服务器的日志一天天地变大，这样子下去的话容易出现硬盘告警，于是想到了使用一些方式去定期清空和压缩一下旧的日志，网络查找一段时间后，发现`logrotate`是不错的选择。

![-w982](/assets/images/15183212133190.jpg)

这个配置有几个要点：

`daily`: 每天执行一次
`dateext`: 备份的日志按日期后缀来保存
`compress`: 压缩日志（这个很有用，往往可以把几GB的压缩成十多MB的文件
`copytruncate`: 备份日志时不会影响使用它的程序的使用，可以持续往原有的文件描述符里边输出内容

立即执行：

```sh
logrotate -v -f /etc/logrotate.d/myconf
```

调试输出：（仅打印不执行任何动作）

```sh
logrotate -d -f /etc/logrotate.d/myconf
```




参考资料：
1. https://huoding.com/2013/04/21/246
2. https://www.thegeekstuff.com/2010/07/logrotate-examples/


