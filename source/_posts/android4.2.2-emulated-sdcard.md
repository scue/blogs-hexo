---
layout: post
date: 2015-08-19 00:00:00
title: "Android 4.2.2 Emulate Sdcard"
description: ""
category: 技术
tags: 
  - tag
---

参考链接: [source.android.com](https://source.android.com/devices/storage/config-example.html)

由于官方提供的参考方法不能使用, 所以我把修改的配置贴了出来

### init.hardware.rc

    on post-fs-data
        mkdir /data/media 0770 media_rw media_rw
        chown media_rw media_rw /data/media

    on init
        mkdir /mnt/shell/emulated 0771 system sdcard_rw
        mkdir /mnt/shell/test 0771 system sdcard_rw
        mkdir /storage/emulated 0775 root root

        export EXTERNAL_STORAGE /storage/emulated/0
        export EMULATED_STORAGE_SOURCE /mnt/shell/emulated
        export EMULATED_STORAGE_TARGET /storage/emulated

        # Support legacy paths
        symlink /storage/emulated/0 /sdcard
        symlink /storage/emulated/0 /mnt/sdcard
        symlink /storage/emulated/0 /storage/sdcard0
        symlink /mnt/shell/emulated/0 /storage/emulated/0

    service sdcard /system/bin/sdcard /data/media /mnt/shell/emulated 1023 1023
        disabled

    on property:dev.bootcomplete=1
        start sdcard


### storage_list.xml

    <storage
            android:storageDescription="@string/storage_internal"
            android:emulated="true"
            android:mtpReserve="100" />


## 实际操作

由于我希望把 `mtd@user` 这个分区挂载为 ext4 格式，以减少使用FAT32格式导致文件系统不稳定的引发的问题；

所以，我总体思路是，把 `mnt@user` 以 ext4 文件系统格式挂载到 `/mnt/user`；

然后再使用 `/system/bin/sdcard` 模拟一个SDcard出来至 `/mnt/sdcard/emulated`；

最后再软链接到 `/storage/emulated` 和 `/mnt/sdcard`;

### init.rk30board.rc

    on fs
        mkdir /mnt/user 0770 media_rw media_rw
        mount ext4 mtd@user /mnt/user wait noatime nodiratime nosuid nodev noauto_da_alloc

    on post-fs
        chown media_rw media_rw /mnt/user

    on post-fs-data
        # we will remap this as /mnt/sdcard with the sdcard fuse tool
        mkdir /data/media 0770 media_rw media_rw
        chown media_rw media_rw /data/media

    on init

        mkdir /mnt/shell/emulated 0771 system sdcard_rw
        mkdir /mnt/shell/test 0771 system sdcard_rw
        mkdir /storage/emulated 0775 root root

        export EXTERNAL_STORAGE /storage/emulated/0
        export EMULATED_STORAGE_SOURCE /mnt/shell/emulated
        export EMULATED_STORAGE_TARGET /storage/emulated

        # Support legacy paths
        symlink /storage/emulated/0 /sdcard
        symlink /storage/emulated/0 /mnt/sdcard
        symlink /storage/emulated/0 /storage/sdcard0
        symlink /mnt/shell/emulated/0 /storage/emulated/0

        # create virtual SD card at /mnt/sdcard, based on the /data/media directory
        # daemon will drop to user/group system/media_rw after initializing
        # underlying files in /data/media will be created with user and group media_rw (1023)
        service sdcard /system/bin/sdcard /mnt/user /mnt/shell/emulated 1023 1023
           disabled

        # for emulated sdcard
        on property:dev.bootcomplete=1
            start sdcard

### storage_list.xml

    <storage
            android:storageDescription="@string/storage_internal"
            android:emulated="true"
            android:mtpReserve="100" />
