---
layout: post
date: 2015-05-07 00:00:00
title: "课程2: Android单元测试"
description: ""
category: 课程
tags: 
  - 
---
# Android单元测试

## 	Java单元测试：

*	解决的问题：使用单元测试，对一个方法进行测试，通常包括 正确性、错误性、边界性等测试，从源头上解决缺陷引入；

*   使用方法：这只是一个Demo，我们部门还没有得到有力的推广（敏捷测试理论上更需要单元测试，但我们项目时间都比较紧）；

        ./junit_gen.sh <项目目录> # 然后填写测试代码到Test.java
        ./test_on_android.sh

*   详情请参考： [链接](http://200.200.0.36/28120/junit_test_android/tree/linux)

####Java覆盖率报告：

*   解决的问题：看看到底我们测试到了哪里，哪些我们覆盖到了？

*   使用方法：emma，编译时应该输入参数 ant emma debug，这由Android SDK已实现；

*   详情请参考： [链接](http://200.200.0.36/28120/junit_test_android/blob/linux/test_on_android.sh#L112)

####Java单测总体报告：

*   解决的问题：现阶段的单元测试（Junit）并没有生成一份Jenkins可识别的报告，这里是解决Jenkins识别不到的问题；
*   详情请参考： [链接](http://200.200.0.36/28120/junit_test_android/blob/linux/android-junit-report-1.5.8.jar)

####Java单测 - 效果展示：

    [Java单测效果](http://200.200.0.36/28120/junit_test_android/tree/linux/ScreenShot)
    
##	NDK单元测试：（C/C++）

*	解决的问题：与Java单元测试类似，但是针对的是NDK（C/C++），NDK的一个明显的好处是它的安全性要比Java的好；

*	使用方法：这是一个简单的Demo

    -	注意到 `sangfor_android`目录
    -	Jni代码放置到 `sangfor_android/jni`
    -	测试代码添加到 `demo_unit.cpp`
    -	然后执行 `./run_linux.sh` 即可

*	详情请参考： [链接](http://200.200.0.36/28120/gtest_android_ndk/tree/all_in_one)

*   注意事项：采用是gtest，它提供了很多测试实例，我们可以去学习一下 [Gtest模板](http://200.200.0.36/28120/gtest_android_ndk/tree/all_in_one/gtest-1.7/samples)

####NDK覆盖率报告：

*   解决的问题：覆盖率报告可以让我们清楚的知道，哪一行代码被执行了多少次

*   使用方法：`./gen_gcov_html.sh <源码目录>`

        ./gen_gcov_html.sh sangfor_android

*   详情请参考： [链接](http://200.200.0.36/28120/gtest_android_ndk/blob/all_in_one/gen_gcov_html.sh)

## 创新分享：

主要讲解一下思路来源，拿到问题之后怎么思考去解决它

分享一下思维，不具体深入讲解如何实现（具体实现包含在创新文章里边了）。
