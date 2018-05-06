---
layout: post
title: "手把手教你交叉编译zip（for Android/Linux）"
description: "静态编译，支持Android和Linux"
category: 技术
tags: [zip, Android, Linux]
---

# 背景

最近想在用户的磁盘上加密一点东西，不算特别重要，但又不期望用户看得到里边的内容，觉得使用zip还不错，可以直接进行一些简单的加密，So，动手干吧，移植它。

# 如处获取源码

我找了一下github没有找到，于是想到ubuntu有：

```sh
type zip
dpkg -S /usr/bin/zip
apt-get source zip # 获取zip源码
```

![](https://ws1.sinaimg.cn/large/6e22ca27gy1fr20q9qjtlj20ia0cnq7s)



值得注意的是，如果提示了需要配置`deb-src`的话就配置一下，然后再使用`apt-get update`更新一下。

输出的文件目录是 `zip-3.0/`

# 进入交叉编译环境

```sh
docker run --rm -it -v $PWD:/work dockcross/linux-armv7 bash
```

修改一下makefile文件`vim unix/Makefile`，将里边的`CC`和`CPP`修改为交叉编译工具链：

![](https://ws1.sinaimg.cn/large/6e22ca27gy1fr20qmoad5j20s70nojw8)


> 考虑到需要将这个二进制程序放到不同的平台（如Android/Linux嵌入式），于是将它静态编译比较容易移植一些。

# 开始编译

执行命令：`make -f unix/Makefile -j8 generic`

![](https://ws1.sinaimg.cn/large/6e22ca27gy1fr20s1tq00j20vt0p0wt4)


# 使用方式


![](https://ws1.sinaimg.cn/large/6e22ca27gy1fr20sajuolj20jn0mtdmg)


