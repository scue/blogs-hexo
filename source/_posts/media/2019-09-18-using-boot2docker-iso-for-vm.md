
---
layout: post
title: "使用boot2docker.iso来搭建一个超小的VM虚拟机环境"
description: ""
category: 技术
tags: [docker, boot2docker, vbox]
---

使用**[boot2docker.iso](https://github.com/boot2docker/boot2docker)**搭建超小VM有几个好处：
1. 做出来的镜像非常小，大概只有不到100MB（相比之下ubuntu则有500MB辣么大），非常好进行分发
2. 根文件系统某种意义上说是「只读」的，无论对根文件系统做了什么，重启就还原了
3. 我们场景合适使用docker来提供服务，并放置这个VM内，打包给客户~
4. 同时业务需要升级的话，只需要对镜像进行升级即可~ ;-) 

<!-- more -->

## 前序工作
a. 镜像下载：
https://github.com/boot2docker/boot2docker/releases

b. 创建虚拟机：
```sh
docker-machine create \
  --driver virtualbox \
  --virtualbox-boot2docker-url=/Users/scue/Downloads/boot2docker.iso \
  magicvm
```

c. 导入镜像：
```sh
eval $(docker-machine env magicvm)
docker load -i magic_business-0.1.1.tar.gz # 这是承载我们业务的镜像
```

此时，我们可以在VBOX的界面上，看到一个`magicvm`虚拟机了，接下来就可以对这个VM进行定制了。


## 1）网卡配置
默认有两张网卡，我们只使用其中一个
a, 把 Adapter 2设定为'Not Attached'，取消勾选
b. 把 Adapter 1设定为'Bridged Adapter'，选择需要桥接的网卡

## 2）共享目录
默认情况下会有一个共享目录，我们要去删除它

## 3）设定docker用户密码（临时）
`sudo passwd docker`，输入新密码，为后续的SSH远程登录使用

## 4）远程登录
```sh
# 为本机网卡添加一个IP地址
ip addr add 192.168.1.33/24 dev en0
# SSH远程登录VM
sshpass -p docker ssh docker@192.168.1.231
```


## 5）设定VM的hostname
假定现在需要设定为 `magic`
```sh
echo magicgw > /var/lib/boot2docker/etc/hostname
```
这个配置重启生效

## 6）设定静态IP地址
有一个配置文件`/var/lib/boot2docker/profile`是在docker启动过程中`/etc/init.d/docker`调用的
我们可以直接修改这个配置文件以达到修改静态IP地址的目的（或许也会其他更好的方式）
往`/var/lib/boot2docker/profile`添加一段配置：
```sh
# setting ip address if need
/sbin/ifconfig eth0 192.168.1.231 netmask 255.255.255.0 up
/usr/local/sbin/ip route add default via 192.168.1.1 || \
 /usr/local/sbin/ip route change default via 192.168.1.1
```

## 7）定制userdata.tar
由于系统目录是由`boot2docker.iso`并且最终是一个`tmpfs`文件系统，重启之后啥也没有了
我们刚刚修改的`/var/lib/boot2docker`实际是一个软链接到已挂载的硬盘上：
```sh
$ ls -l /var/lib/
total 0
lrwxrwxrwx 1 root root 29 Sep 11 03:03 boot2docker -> /mnt/sda1/var/lib/boot2docker
lrwxrwxrwx 1 root root 24 Sep 11 03:03 docker -> /mnt/sda1/var/lib/docker
drwxr-xr-x 4 root root 160 Sep 11 03:03 nfs
drwxr-xr-x 2 root root 40 Sep 4 16:15 sshd
```
开机启动过程中，会把`/var/lib/boot2docker/userdata.tar`的文件内容解压至 `/home/docker`，以实现SSH登录授权。
由于存在这个机制，我们也可以利用它，定制一下HOME目录，以实现快捷访问定制化的内容：
```sh
mkdir -p /mnt/sda1/magic # 只有/mnt/sda1的内容才会被保存（其他位置重启失效）
cd ~
ln -svf /mnt/sda1/magic magic # 创建一个软链接
tar rf /var/lib/boot2docker/userdata.tar magic # 将此软链接添加至 userdata.tar 文件
```
这样，下次启动的进入VM时，就可以通过 `ls` 目录看到HOME目录下有一个`magic` 目录的软链接了，以快捷访问此目录。


## 8）导出VM
导出VM不能直接使用VBOX的界面进行导出，否则会把iso文件给遗漏了，使用命令行可以解决这个问题：

```sh
VBoxManage export magicvm -o magicvm.ova --iso
```

OK, 这一个虚拟机VM到此为止就做好了。~