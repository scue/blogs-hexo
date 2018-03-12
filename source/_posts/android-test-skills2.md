---
layout: post
date: 2015-04-16 00:00:00
title: "课程1: Android测试技巧(二)"
description: ""
category: 课程
tags: 
  - 课程
---
# Bug定位技巧

今天比较匆忙，准备得不算很多；

首先来针对一些普通的Bug来分析一下如何定位一个Bug的问题；

一些方法运用好，可以更好的重现一些难以重现的问题，并或许还可以提供有效的Bug解决方法。

1.	日志输出，依据logcat输出的**进程号**来定位问题（如EasyConnect，以下截图仅举例）：

    ![logcat-pid](/assets/Courses/Courses01_android_test_skills_1.png)

2.	日志输出，依据Logcat输出的**详细时间**来定位问题（如发现问题的那一分钟内、以下截图仅举例）：

    ![logcat-time](/assets/Courses/Courses01_android_test_skills_2.png)

3.	输出报告，adb bugreport

    ![adbBugReport](/assets/Courses/Courses01_android_test_skills_3.png)

4.	善于使用索引工具，如OpenGrok

	外网可以参考：[AndroidXRef](http://androidxref.com)

    内网可以参考：[AndroidOS4C](http://200.200.137.175:8080/source), 内网有权限控制

    ![AndroidXRef](/assets/Courses/Courses01_android_test_skills_4.png)
