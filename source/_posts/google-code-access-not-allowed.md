---
layout: post
date: 2015-05-28 00:00:00
title: "Google Code Access Not Allowed 解决方法"
description: ""
category: 技术
tags: 
  - Google Code
---

&emsp;&emsp;很久以前，还年轻，不清楚Google Code是不允许托放一些文件（即当网盘来使用） 导致后来Google Code帐号被不知情的情况下，被封了，每次想打开code.google.com上的项目时，要以隐身模式打开，或是退出登录，或是用其他帐号来访问，感觉非常麻烦。

&emsp;&emsp;想到这与cookies有关系，于是乎就下手去查了一下Cookies，发现没有几个Cookies，很快就定位到了与 NID的值有关联。通过Google Chrome 应用商店，找到 [EditThisCookie](http://www.editthiscookie.com/){:target="brank"}，新增屏蔽规则：

|      域名     |  名称  |   值   |
|    --------   |--------|--------|
|code.google.com|   NID  |   any  |
