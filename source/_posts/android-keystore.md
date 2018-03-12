---
layout: post
date: 2015-01-18 00:00:00
title: "Android Keystore 那些事儿"
description: ""
category: 技术
tags: 
    - Android
    - Keystore
---

## 生成Keystore

一键生成 Keystore

    keytool -genkey -v -keystore scue15K.keystore -alias scue15K \
        -keyalg RSA -keysize 2048 -validity 15000 \
        -keypass scue15 -storepass scue15 \
        -dname 'CN=scue.github.io, OU=SDET, O=Android, L=ShenZhen, S=GuangDong, C=CN'

得到`scue15K.keystore`这个文件，keypass和storepass密码都是scue15

## 检验Keystore

使用一行命令检查Keystore信息

    keytool -list -v -keystore scue15K.keystore # 需键入 storepass

## 使用Keystore签名

重新签名的步骤，同样适用于正常签名

    zip -d in.apk 'META-INF/*' # 先清理旧的签名，假如你是重新签名的话
    jarsigner -verbose -sigalg SHA1withRSA -digestalg SHA1 -keystore /tmp/scue15K.keystore \
        -keypass scue15 -storepass scue15 -signedjar out.apk in.apk scue15K
    zipalign -v 4 out.apk release.apk 
