---
layout: post
date: 2015-04-28 00:00:00
title: "Android字符串资源检查"
description: ""
category: 技术
tags: 
  - Android
  - Python
---

Android项目开发过程中，容易出现缺少对应中英文翻译的情况，这个Python脚本是用于检查字符串是否缺少了对应的翻译

脚本链接：

[Android字符串资源检查](htps://gist.github.com/scue/0ad950803038d9eab634)

使用方法：

    ./check_string_res.py packages/apps/Settings/
    ./check_string_res.py packages/apps/Settings/ packages/apps/QuickSearchBox/ ..

效果如下：

    $ ./check_string_res.py packages/apps/Bluetooth/

    ###正在检查项目packages/apps/Bluetooth/

    >>> Checking res/values/strings.xml file ..
      - Warning: string name 'auth_notif_message' not found!!!
      - Warning: string name 'auth_notif_ticker' not found!!!
      - Warning: string name 'auth_notif_title' not found!!!
      - Warning: string name 'cancel' not found!!!
      - Warning: string name 'defaultname' not found!!!
      - Warning: string name 'localPhoneName' not found!!!
      - Warning: string name 'ok' not found!!!
      - Warning: string name 'pbap_authentication_timeout_message' not found!!!
      - Warning: string name 'pbap_session_key_dialog_header' not found!!!
      - Warning: string name 'pbap_session_key_dialog_title' not found!!!
      - Warning: string name 'unknownName' not found!!!

    >>> Checking res/values-zh-rCN/strings.xml file ..
      - Warning: string name 'auth_notif_message' not found!!!
      - Warning: string name 'auth_notif_ticker' not found!!!
      - Warning: string name 'auth_notif_title' not found!!!
      - Warning: string name 'bluetooth_share_file_name' not found!!!
      - Warning: string name 'cancel' not found!!!
      - Warning: string name 'defaultname' not found!!!
      - Warning: string name 'localPhoneName' not found!!!
      - Warning: string name 'ok' not found!!!
      - Warning: string name 'pbap_authentication_timeout_message' not found!!!
      - Warning: string name 'pbap_session_key_dialog_header' not found!!!
      - Warning: string name 'pbap_session_key_dialog_title' not found!!!
      - Warning: string name 'unknownName' not found!!!
