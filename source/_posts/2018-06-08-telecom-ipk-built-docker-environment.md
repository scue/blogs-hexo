---
layout: post
title: "电信光猫插件编译环境Docker使用文档"
description: ""
category: 技术
tags: [Telecom, IPK]
---

# 镜像安装
- 镜像下载链接：https://dlupdate.onethingpcs.com/dianxin/dianxin_sdk_built.img.tgz 
- MD5校验码：785ab9973be55f2b8930f122eb3777cf
- 下载完成解压：`tar zxvf dianxin_sdk_built.img.tgz` → `dianxin_sdk_built.img`
- 将镜像导入：`docker load -i dianxin_sdk_built.img`
- 导入之后可以看到镜像：`dianxin:sdk-built`

# 进入镜像
- 假定你的代码在当前目录，比如我的是`$PWD/data_collect`
- `docker run --rm -it -v $PWD:/work dianxin:sdk-built bash`

  ![](https://media-1256569450.cos.ap-chengdu.myqcloud.com/blog/15284399779145.jpg)

- `make menuconfig` 选择 → `Upointech` → `data_collect` （按空格选择）

  ![](https://media-1256569450.cos.ap-chengdu.myqcloud.com/blog/15284401221191.jpg)

- `make package/data_collect/{clean,compile} V=s`执行编译

  ![](https://media-1256569450.cos.ap-chengdu.myqcloud.com/blog/15284402644093.jpg)

- 输出的IPK文件在`bin/`目录下，使用`find`查找比较方便一些

# 关于DataCollect

项目源码：http://wx-gitlab.xunlei.cn/lingweiqiang/Telecom-Gateway-Plugin-data-collect



