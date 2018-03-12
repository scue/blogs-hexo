---
layout: post
date: 2015-05-09 00:00:00
title: "Eclipse ADT定制"
description: ""
category: 技术
tags: 
  - Android
  - Eclipse
---

1.  配置自动补全：

    Windows -> preferences -> 搜索assist，修改 java xml自动触发补全：

        .abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ_

2.  自动补全插件：

    [Eclipse Tab自动补全](http://www.cnblogs.com/sunjie21/archive/2012/06/28/2567463.html)

    Windows -> Preference -> 搜索 Assist，C/C++、Java、XML都输入（或追加）：

        .abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ_

    直接替换文件方法：

        # 备份文件
        find eclipse/plugins -name \*jface.text\* -exec mv -v {} {}.bak \;
        # 复制文件到
        eclipse/plugins/org.eclipse.jface.text_3.8.2.v20121126-164145.jar #(要相同版本才可以)

3.  修改边框过大以及颜色配置：

    3.1.   修改提示框背景颜色：

    路径：/usr/share/themes/Greybird/gtk-2.0/gtkrc 修改：

        gtk-color-scheme    = "tooltip_bg_color:#f2edbc\ntooltip_fg_color:#000000" # Tooltips.

    3.2.   修改全局背景颜色：

        gtk-color-scheme    = "bg_color:#cce8cf\nselected_bg_color:#398ee7\nbase_color:#fcfcfc" # Background, base.

    3.3.   修改eclispe背景颜色：

        windows->Preferences->General->Editor->Text Editors->Backgroud color修改成#fcfcfc

    3.4.   边框过大的调整方法

        cd /path/to/adt
        vi "$(find eclipse -name e4_default_gtk.css)"
        # 打开： plugins/org.eclipse.platform_4.2.*/css/e4_default_gtk.css，找到并修改：
         .MPartStack {
            font-size: 9;
            font-family: Liberation Sans;
            swt-tab-renderer: null;
            swt-tab-height: 22px;
            swt-selected-tabs-background: #FFFFFF #ECE9D8 100%;
            swt-simple: false;
            swt-mru-visible: false;
         }
        # 原始内容：
        .MPartStack {
            font-size: 11;
            swt-simple: false;
            swt-mru-visible: false;
        }

    3.5.   Logcat 字体大小： Android → Logcat位置

4.  崩溃解决方法：

    打开 eclipse/configuration/config.ini 文件， 在最后一行添加

        org.eclipse.swt.browser.DefaultType=mozilla

    参考博客： [Eclipse崩溃总结](http://www.cnblogs.com/scue/p/4461845.html)

5.  Eclipse快捷键：

    vi ~/.local/share/applications/eclipse.desktop 

    Sample1:

        [Desktop Entry]
        Type=Application
        Name=Eclipse
        Comment=Eclipse Integrated Development Environment
        Icon=eclipse.png
        Exec="/media/Study/Program Files/adt-bundle-linux-x86_64-20131030/eclipse/eclipse"
        Terminal=false
        Categories=Development;IDE;Java;
        StartupWMClass=Eclipse

    Sample2: for `ubuntu 15.04`

        [Desktop Entry]
        Encoding=UTF-8
        Version=1.0
        Name=Eclipse
        Comment=Eclipse IDE
        Type=Application
        Exec=env JAVA_HOME=/home/sinfor/apt/jdk1.7.0_67 /media/Study/android/eclipse/eclipse_luna_scue/eclipse
        Icon=/home/sinfor/.local/share/icons/adt.png

