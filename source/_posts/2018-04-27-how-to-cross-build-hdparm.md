---
layout: post
title: "手把手教你交叉编译hdparm"
description: "HDPARM是一个磁盘测速工具"
category: 技术
tags: [hdparm]
---



- 相关背景
	- Hdparm是一个磁盘测速工具
	- 官网：https://sourceforge.net/projects/hdparm/
- 源码下载
	- git clone https://github.com/Distrotech/hdparm.git
- 开始编译
	- 进入交叉编译环境：docker run --rm -it -v $PWD:/work dockcross/linux-armv7 bash
	- 设置环境变量：export CFLAGS=-static; export STRIP=arm-linux-gnueabihf-strip
	- 修改LDFLAGS: vim Makefile，加上 -static以静态链接
	- ![](https://ws1.sinaimg.cn/large/6e22ca27gy1fqr4eyxobmj20jb0bhmz1)
	- make -j8
	- ![](https://ws1.sinaimg.cn/large/6e22ca27gy1fqr4f719q7j20vt0p0wrt)

- 验证工具
	- Android:
		- adb push hdparm /data/local/tmp/
		- adb shell /data/local/tmp/hdparm -t -T --direct /dev/block/system_a
	- Linux Arm:
		- /tmp/hdparm -t -T --direct /dev/sda


