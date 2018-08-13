---
layout: post
title: "Docker远程机器的使用"
description: ""
category: 技术
tags: [docker]
---

简而言之：在本地环境中使用docker，宿主机是远程机器

由于最近我的Mac Pro坏掉拿去维修了，不得不使用我的小小的Air来办公，然而它只有4GB内存，远远无法满足我的开发需求，开一个Chrome浏览器和一个Intellij Idea它就卡得不行了。

又由于我的工作性质原因，我需要docker来使用mysql、redis服务器，交叉编译环境等等。

于是想到，能不能将docker的宿主机运行到某一台不使用的台式机上。

答案：当然可以。

<!-- more -->

# 开始配置

目标机器是一台闲置的ubuntu环境，我就命名为`ubuntuengine`吧。

首先，在目标机器开启root权限登录：

修改文件：`vim /etc/ssh/sshd_config`

```conf
PermitRootLogin yes
```

执行：`systemctl restart ssh`重启sshd


在mac环境上执行以下命令：

```sh
ssh-copy-id root@ubuntuengine
docker-machine create --driver generic --generic-ip-address=192.168.118.53 --generic-ssh-key ~/.ssh/id_rsa --generic-ssh-user root ubuntuengine
```

随后，我们使用`docker-machine env ubuntuengine`查看远程机器的信息：

```txt
$ docker-machine env ubuntuengine
export DOCKER_TLS_VERIFY="1"
export DOCKER_HOST="tcp://192.168.118.53:2376"
export DOCKER_CERT_PATH="/Users/scue/.docker/machine/machines/ubuntuengine"
export DOCKER_MACHINE_NAME="ubuntuengine"
# Run this command to configure your shell:
# eval $(docker-machine env ubuntuengine)
```



使之生效的方式：



```sh
eval $(docker-machine env ubuntuengine)
```

![](https://media-1256569450.cos.ap-chengdu.myqcloud.com/blog/15341441230435.jpg)


我们来尝试玩一下：

```sh
docker run --rm -it -v /tmp:/tmp alpine sh
```

是不是感觉好好玩呢~


参考链接：
1. https://www.kevinkuszyk.com/2016/11/28/connect-your-docker-client-to-a-remote-docker-host/ 
2. https://docs.docker.com/machine/reference/create/
https://docs.docker.com/machine/reference/mount/
