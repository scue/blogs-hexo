---
layout: post
title: "聊一聊Android APK系统签名~"
description: ""
category: 技术
tags: [Android]
---

# 通过signapk.jar签名


```sh
java -Xmx2048m -Djava.library.path=prebuilts/sdk/tools/linux/lib64/ \
    -jar out/host/linux-x86/framework/signapk.jar \
    -w build/target/product/security/platform.x509.pem \
    build/target/product/security/platform.pk8 \
    /tmp/app-debug.apk /tmp/app-signed.apk
```

<!-- more -->

对java的版本要求较高，这边使用的是Java 8：

```txt
$ java -version
openjdk version "1.8.0_141"
OpenJDK Runtime Environment (build 1.8.0_141-8u141-b15-3~14.04-b15)
OpenJDK 64-Bit Server VM (build 25.141-b15, mixed mode)
```

不然会提示：

```txt
Exception in thread "main" java.lang.UnsupportedClassVersionError: com/android/signapk/SignApk : Unsupported major.minor version 52.0
```

同时`-Djava.library.path=prebuilts/sdk/tools/linux/lib64/`也是必须的，不然会提示：

```txt
Exception in thread "main" java.lang.UnsatisfiedLinkError: no conscrypt_openjdk_jni in java.library.path
```

> 提示：如果内部其他团队经常使用到这个系统签名的功能，可以在内部开发一个签名服务器。
> ![](https://media-1256569450.cos.ap-chengdu.myqcloud.com/blog/15287105859218.jpg)

# 通过Android.mk签名

一个`Android.mk`的示例：

```makefile
LOCAL_PATH := $(call my-dir)

LOCAL_PATH:= $(call my-dir)
include $(CLEAR_VARS)
LOCAL_MODULE := DataCollect
LOCAL_MODULE_TAGS := optional
LOCAL_SRC_FILES := app-debug.apk
LOCAL_MODULE_CLASS := APPS
LOCAL_MODULE_SUFFIX := $(COMMON_ANDROID_PACKAGE_SUFFIX)
LOCAL_CERTIFICATE := platform
include $(BUILD_PREBUILT)
```

> 提示：若团队有多个类似的APK，可以写成模板。

## 预编译模板

```makefile
define PREBUILT_template
    LOCAL_MODULE:= $(1)
    LOCAL_MODULE_CLASS := APPS
    LOCAL_MODULE_SUFFIX := $$(COMMON_ANDROID_PACKAGE_SUFFIX)
    LOCAL_CERTIFICATE := platform
    LOCAL_SRC_FILES := $$(LOCAL_MODULE).apk
    LOCAL_REQUIRED_MODULES := $(2)
    include $$(BUILD_PREBUILT)
endef

define PREBUILT_APP_template
    include $$(CLEAR_VARS)
    LOCAL_MODULE_TAGS := optional
    $(call PREBUILT_template, $(1), $(2))
endef

prebuilt_apps := \
    otaupdater \
    ServiceBridge \
    DataCollect \
    
$(foreach app,$(prebuilt_apps), \
    $(eval $(call PREBUILT_APP_template, $(app),)))

# ...
```

然后将`DataCollect.apk`放置于有这个`Android.mk`的目录下即可。

