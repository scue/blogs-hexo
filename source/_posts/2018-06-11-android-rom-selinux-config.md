---
layout: post
title: "Android二进制程序的SElinux配置"
description: ""
category: 技术
tags: [Android, SElinux]
---

你知道Android SElinux的权限配置吗？

在Android SElinux配置里边是没有声明就表示没有权限。

没有权限，就会可能导致你的二进制程序的运行效果不如你所愿。

本文章提供了一些技巧，让你方便地配置二进制程序所需要的权限。

<!-- more -->

# 先让二进制跟随init进程启动

找到一个合适的`init.xxx.rc`添加自己的配置：
```
#data_collect
service data_collect /system/bin/logwrapper /system/bin/data_collect -c /system/etc/cfg.data_collect.json
    class main
    user root
    disabled
    seclabel u:r:data_collect:s0

on property:dev.bootcomplete=1
    start data_collect
```

假定我的二进制程序是`data_collect`。

- 这个service的名字是`data_collect`
- 在开机启动完成之后，自动拉起二进制`/system/bin/data_collect`
- 参数是`-c /system/etc/cfg.data_collect.json`
- 其中`/system/bin/logwrapper`是为了将`stdout`, `stderr`重定向到logcat输出
- `seclabel u:r:data_collect:s0`配置与selinux有关，相关的配置文件是`data_collect.te`文件

# 写一个空壳的Selinux配置文件

**此时此刻，我们实际上不并不知道它需要哪些权限**。

无妨，先让它运行起来，然后再使用日志去收集它哪些被deny了。

写一个空壳的配置文件`device/amlogic/common/sepolicy/data_collect.te`，让它运行起来：

```
type data_collect, domain;
type data_collect_exec, exec_type, file_type;
init_daemon_domain(data_collect)

```

# 收集运行日志

## 收集方式

`adb logcat -d | grep data_collect | grep avc >/tmp/avc_data_collect.txt`

## 解析日志

我们将`avc_data_collect.txt` push到编译环境，然后使用 `external/selinux/prebuilts/bin/audit2allow -i /tmp/avc_data_collect.txt` 得到所需要的权限


![](https://media-1256569450.cos.ap-chengdu.myqcloud.com/blog/15287168107577.jpg)

把输出的结果写到`data_collect.te`文件里边，这个收集过程是持续的，并不是一下子可以把所有功能都会运行到。

# 编译重新烧写生效

改了之后还需要重新编译一下boot.img

```
. build/envsetup.sh
lunch 30 # 依赖于你的配置
make -j32 bootimage
```

烧写方式：`flash-boot.sh`

```
adb connect 192.168.113.81
adb push $OUT/boot.img /data/local/tmp/
adb shell su root dd if=/data/local/tmp/boot.img of=/dev/block/boot_a
adb shell su root reboot
```


## 编译可能遇到的问题

### 问题1：`*** No X.509 certificates found **`

如果遇到`common/kernel/Makefile:146: *** No X.509 certificates found **`
解决方式：`rm -rf out/target/product/onething_onecloudpro/obj/KERNEL_OBJ/ out/target/common/`



### 问题2： `1 neverallow failures occurred`

```
libsepol.report_failure: neverallow on line 652 of system/sepolicy/domain.te (or line 9450 of policy.conf) violated by allow data_collect fuse_device:chr_file { getattr };
libsepol.check_assertions: 1 neverallow failures occurred
```

此时需要修改一下`system/sepolicy/domain.te`配置文件，在里边相应的位置上添加上`-data_collect`。




