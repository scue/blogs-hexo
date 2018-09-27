---
layout: post
title: "修复MacOS的Matplotlib中文乱码问题"
description: ""
category: 技术
tags: [macos, matplotlib]
---

最近发现MacOS使用`Matplotlib`的时候中文显示乱码，只需要花1分钟配置一下，可以解决这个问题。

<!-- more -->

## 第一步，查看python的路径
```sh
$ type python
python is /usr/local/anaconda3/envs/py27/bin/python
```

> 提示：我这里边使用的是`anaconda`来管理多个Python版本信息

## 第二步，拷贝`SimHei.ttf`至指定目录，及配置`matplotlibrc`文件

```sh
$ cd /usr/local/anaconda3/envs/py27/lib/python2.7/site-packages/matplotlib/mpl-data/fonts/ttf

$ cp ~/Downloads/simhei.ttf  SimHei.ttf

$ cd ../..

$ tree 
├── fonts/
├── images/
├── stylelib/
└── matplotlibrc
```

`vim matplotlibrc`在最后添加上

```rc
font.family         : sans-serif
font.sans-serif     : SimHei
axes.unicode_minus  : False
```

## 第三步，删除缓存的ttf

```sh
rm -rf ~/.matplotlib/*.cache
rm ~/.matplotlib/fontList.json // 必须执行这一行，否则不生效
```

最后运行一下，发现问题解决了。

参考链接：
1. [彻底解决matplotlib中文乱码问题](https://blog.csdn.net/dgatiger/article/details/50414549)
2. [matplotlib font not found](https://stackoverflow.com/questions/26085867/matplotlib-font-not-found)
3. [黑体字体simhei.ttf](http://www.font5.com.cn/zitixiazai/1/151.html)