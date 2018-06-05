---
layout: post
title: "gomobile入门指南"
description: "GO语言支持移动应用开发，本文章主要考虑Android移动应用开发，你可以将本文章做为一个指南。"
category: 技术
tags: [GO, gomobile]
---


# 初始化环境

官方的WiKi链接：https://github.com/golang/go/wiki/Mobile ，但我感觉它写得不是很好，尤其是`gomobile init`的部分。

首先，我安装了Android Studio，安装好之后，默认的SDK路径是`/Users/scue/Library/Android/sdk`。

其次，我安装了Android NDK，安装好之后，我这边的路径是`/Users/scue/source/Android/android-ndk-r16b`。

随后，我们依次执行以下命令来安装`gomobile`和初始化`gomobile`：

```go
gopm get -g -v -d golang.org/x/mobile
gopm bin -v -d ~/go/bin golang.org/x/mobile/cmd/gomobile
export ANDROID_HOME=/Users/scue/Library/Android/sdk
gomobile init -ndk /Users/scue/source/Android/android-ndk-r16b
```

> `gopm`是一个go语言的包管理工具，你可以通过搜索本博客找到如何安装和使用它。

# 尝试运行示例

```go
gopm get -g -v -d golang.org/x/mobile/example/basic
gomobile build -target=android golang.org/x/mobile/example/basic
```

这样子就会在`~/go/src`目录下生成一个 `basic.apk` 文件，我们就可以直接使用`adb install basic.apk`来安装它。当然，官网还提供了一个更简便的方式：

```sh
gomobile install golang.org/x/mobile/example/basic
```

它不但编译好了，还直接帮忙安装并运行起来了。此时此刻，如果你想手动再拉起来的话，可以这样子：

```sh
adb shell am start -n org.golang.todo.basic/org.golang.app.GoNativeActivity
```

应用的包名是`org.golang.todo.basic`，Activity是`org.golang.app.GoNativeActivity`。我仔细看过gomobile的传入参数，没有发现可以设置的package和activity的方法，这相当于差不多已经在gomobile里边写死了。

如果你对`package`和`activity`是固定的还不死心的话，可以通过`gomobile build -v -target=android golang.org/x/mobile/example/basic`看看详细的输出：

```txt
$ gomobile build -v -target=android golang.org/x/mobile/example/basic
generated AndroidManifest.xml:
<?xml version="1.0" encoding="utf-8"?>
<manifest
	xmlns:android="http://schemas.android.com/apk/res/android"
	package="org.golang.todo.basic"
	android:versionCode="1"
	android:versionName="1.0">

	<application android:label="Basic" android:debuggable="true">
	<activity android:name="org.golang.app.GoNativeActivity"
		android:label="Basic"
		android:configChanges="orientation|keyboardHidden">
		<meta-data android:name="android.app.lib_name" android:value="basic" />
		<intent-filter>
			<action android:name="android.intent.action.MAIN" />
			<category android:name="android.intent.category.LAUNCHER" />
		</intent-filter>
	</activity>
	</application>
</manifest>
golang.org/x/mobile/event/paint
image/color
golang.org/x/mobile/app/internal/callfn
golang.org/x/mobile/geom
golang.org/x/mobile/event/touch
golang.org/x/mobile/event/lifecycle
golang.org/x/mobile/event/key
golang.org/x/mobile/exp/f32
golang.org/x/mobile/gl
image
golang.org/x/mobile/event/size
image/internal/imageutil
image/draw
golang.org/x/mobile/exp/gl/glutil
golang.org/x/mobile/app
golang.org/x/mobile/exp/app/debug
golang.org/x/mobile/example/basic
golang.org/x/mobile/event/paint
image/color
golang.org/x/mobile/app/internal/callfn
golang.org/x/mobile/geom
golang.org/x/mobile/event/key
golang.org/x/mobile/event/touch
golang.org/x/mobile/event/lifecycle
golang.org/x/mobile/exp/f32
golang.org/x/mobile/gl
image
golang.org/x/mobile/event/size
image/internal/imageutil
image/draw
golang.org/x/mobile/exp/gl/glutil
golang.org/x/mobile/app
golang.org/x/mobile/exp/app/debug
# golang.org/x/mobile/app
android.c:101:44: warning: incompatible pointer types assigning to 'void (*)(ANativeActivity *, int)' (aka 'void (*)(struct ANativeActivity *, int)') from 'void (ANativeActivity *, GoInt)' (aka 'void (struct ANativeActivity *, long long)') [-Wincompatible-pointer-types]
golang.org/x/mobile/example/basic
golang.org/x/mobile/event/paint
image/color
golang.org/x/mobile/app/internal/callfn
golang.org/x/mobile/event/touch
golang.org/x/mobile/event/lifecycle
golang.org/x/mobile/geom
golang.org/x/mobile/event/key
golang.org/x/mobile/exp/f32
golang.org/x/mobile/gl
image
golang.org/x/mobile/event/size
image/internal/imageutil
image/draw
golang.org/x/mobile/exp/gl/glutil
golang.org/x/mobile/app
golang.org/x/mobile/exp/app/debug
golang.org/x/mobile/example/basic
golang.org/x/mobile/event/paint
golang.org/x/mobile/app/internal/callfn
image/color
golang.org/x/mobile/event/lifecycle
golang.org/x/mobile/geom
golang.org/x/mobile/event/key
golang.org/x/mobile/event/touch
golang.org/x/mobile/exp/f32
golang.org/x/mobile/gl
image
golang.org/x/mobile/event/size
image/internal/imageutil
image/draw
golang.org/x/mobile/exp/gl/glutil
golang.org/x/mobile/app
golang.org/x/mobile/exp/app/debug
# golang.org/x/mobile/app
android.c:101:44: warning: incompatible pointer types assigning to 'void (*)(ANativeActivity *, int)' (aka 'void (*)(struct ANativeActivity *, int)') from 'void (ANativeActivity *, GoInt)' (aka 'void (struct ANativeActivity *, long long)') [-Wincompatible-pointer-types]
golang.org/x/mobile/example/basic
apk: classes.dex
apk: lib/armeabi-v7a/libbasic.so
apk: lib/arm64-v8a/libbasic.so
apk: lib/x86/libbasic.so
apk: lib/x86_64/libbasic.so
apk: AndroidManifest.xml
```

# 别想了，建个Android Studio工程吧

那，怎么设定为自己的程序包名和声明一下程序的权限呢？
没有其他的办法，只好建立一个Android Studio的工程进行开发。

我的建议是基于官网的示例开始，然后慢慢地进行修改，修改的过程使用`git`来管理就好了，很方便。

我们先将官网的示例下载回来：

```sh
go get -d golang.org/x/mobile/example/bind/...
```

官方的Android studio示例具体路径是`golang.org/x/mobile/example/bind/android/`

然后我们将这个工程导入到`Android Studio`

> 提示：导入之前强烈建议使用`git init`先初始化`example/bind/android`，然后再`git commit`一下，这样子就可以记录自己都修改了哪些东西。

随后，我们修改`build.gradle`（app）里边的`applicationId`，将它修改为你自己的应用包名即可。

> 提示：我相信你很可能会遇到`Unable to resolve dependency for ':app@debug/compileClasspath': Failed to transform file 'hello.aar' to match attributes {artifactType=android-exploded-aar} using transform ExtractAarTransform`这个问题，你可以参考我的博客 https://linkscue.com/2018/06/03/2018-06-03-gomobile-problems-set/ 来解决这个问题。

同时，我也建议你将`AndroidManifest.xml`文件的`package="XXX"`也相应的修改一下。

![](https://ws1.sinaimg.cn/large/6e22ca27gy1frz58z2owdj214m0rcqcb)

接下来，你依然还可以使用GO来开发你的各项底层的基础功能，如果你的程序期望一直是后台运行的话，有几点建议：

- 将启动jni层的代码包装成一个Android的Service
- 在App启动时候，判断一下service是否已启动，未启动则拉起Service
- 在App的OnResume()，重复上边的判断，以确保Service被拉起
- 在JNI层所使用到的权限，在AndroidManifest.xml里边也要声明，通常像`INTERNET`和读写外置存储通常是需要的

此外，还有几点建议：

- Android默认程序是可以访问一个data的目录的，一般的在`/data/user/0/<package>/files`
- 一般情况下，这个目录只有你的App可以访问，像(root, system用户)也是可以访问的
- 需要落地的数据可以放置于这里，不过最好落地的数据加密一下
- 如果你的担心你的App权限不足，且你是嵌入式底层App开发的话，建议使用系统签名
- 如果使用了系统签名，需要在AndroidManifest.xml上添加`android:sharedUserId="android.uid.system"`才实际生效
- 系统签名后的UID是1000，可以在程序启动之后`os.Getuid()` 取到

关于安全方面，还有一些建议：

- Android的JNI路径，可以通过Java层上的`getApplicationInfo().nativeLibraryDir`取到
- 一般像这样子： `/data/app/<package>-1/lib/arm64/`
- gomobile编译出来的jni动态库是`libgojni.so`
- 可以基于这个，计算一下自身的md5值，再拿去与服务器比对，以监测自身是否被恶意篡改


