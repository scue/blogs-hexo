---
layout: post
title: "Go语言通过JNI调用java的方法"
description: ""
category: 技术
tags: [go, jni, java]
---

最近有一个问题，是在golang作为底层，想获取apk的版本号和版本号代码（通俗一点就是versionName和versionCode），如果这时java代码，可以很容易地通过

```java
// please add try/catch ... ;-)
PackageInfo info = context.getPackageManager().getPackageInfo(getPackageName(), 0);
String versionName = info.versionName;
int versionCode = info.versionCode;
```

来获取，但是，如果你的主要程序都是go语言来写，需要在go语言中获取apk的版本号怎么处理呢？

<!-- more -->

主要的解决思路有三种：

* 一是在你的go程序初始化的时候，通过java上层传参的形式，将这些参数传递下来
* 二是通过gobind来直接调用java代码
* 三是通过cgo来写ndk代码，调用java方法

# 解决思路的分析

## 第一种java传参给go语言入口函数

优势在于写起来非常简单，只需要go语言的接口加参数即可，然后java调用go语言代码的时候加入参数即可，缺点就是以后有新需求来的时候，还需要加参数，可能参数会越来越长...

## 第二种通过gobind来直接调用java代码

gobind的官方代码有这么一段:

![](https://media-1256569450.cos.ap-chengdu.myqcloud.com/blog/15376617680931.jpg)

一开始我以为我也可以在上层直接将Java代码写好，context什么的直接通过重写Appliction类，得到一个static的context，就可以在go语言中直接使用啦，心里美滋滋的...

但是，直到我将代码都写好了之后，才发现，原来gobind并不支持`import`自己写的类，而是已知的Android API而已...

因此，这种思路尝试到了最后不得不放弃了

> 官网链接：https://godoc.org/golang.org/x/mobile/cmd/gobind

## 第三种通过cgo来调用java代码

可以说，这个方法算是不得以而为之吧，我也是看到了go语言针对asset资源目录的处理方式才有所顿悟：

![](https://media-1256569450.cos.ap-chengdu.myqcloud.com/blog/15376622125423.jpg)

有兴趣的自己翻来看一下，具体路径`$GOPATH/src/golang.org/x/mobile/asset/asset_android.go`。

虽然比较难看，但写起来还是那么酷酷的。

优点是，不需要去动java上层的代码，直接在cgo里边完成需求
缺点是，相对难以理解，对于不了解jni操作的人来说维护起来相对困难，调试也不太方便

# 示例：获取版本号


```go

/*
#cgo LDFLAGS: -landroid
#include <jni.h>
#include <stdlib.h>
#include <string.h>

int versionCode = 0;
char versionName[16] = "";
const char* pVersionName = NULL;

int getVersion(uintptr_t java_vm, uintptr_t jni_env, jobject ctx)
{
    JavaVM *vm = (JavaVM *)java_vm;
    JNIEnv *env = (JNIEnv *)jni_env;
    jclass ctxClazz = (*env)->FindClass(env, "android/content/Context");

    jstring packageName;
    jobject packageManagerObj;
    jobject packageInfoObj;
    jmethodID getPackageName = (*env)->GetMethodID(env, ctxClazz, "getPackageName", "()Ljava/lang/String;");
    jmethodID getPackageManager = (*env)->GetMethodID(env, ctxClazz, "getPackageManager", "()Landroid/content/pm/PackageManager;");

    jclass pmClazz = (*env)->FindClass(env, "android/content/pm/PackageManager");
    jmethodID getPackageInfo = (*env)->GetMethodID(env, pmClazz, "getPackageInfo", "(Ljava/lang/String;I)Landroid/content/pm/PackageInfo;");

    jclass packageInfoClass = (*env)->FindClass(env, "android/content/pm/PackageInfo");
    jfieldID versionCodeFid = (*env)->GetFieldID(env, packageInfoClass, "versionCode", "I");
    jfieldID versionNameFid = (*env)->GetFieldID(env, packageInfoClass, "versionName", "Ljava/lang/String;");

    packageName = (jstring)(*env)->CallObjectMethod(env, ctx, getPackageName);
    packageManagerObj = (*env)->CallObjectMethod(env, ctx, getPackageManager);
    packageInfoObj = (*env)->CallObjectMethod(env, packageManagerObj, getPackageInfo, packageName, 0x0);

    // version code
    versionCode = (*env)->GetIntField(env, packageInfoObj, versionCodeFid);

    // version name
    jstring jstr = (*env)->GetObjectField(env, packageInfoObj, versionNameFid);
    const char *str = (*env)->GetStringUTFChars(env, jstr, NULL);
    strcpy(versionName, str);
    pVersionName = versionName;
    (*env)->ReleaseStringUTFChars(env, jstr, str);

    return versionCode;
}
*/
import "C"

var getVersionOnce sync.Once

func getVersionJni() {
   e := app.RunOnJVM(func(vm, env, ctx uintptr) error {
      C.getVersion(C.uintptr_t(vm), C.uintptr_t(env), C.jobject(ctx))
      return nil
   })
   if e != nil {
      log.Fatalf("getVersionJni: fatal: %s", e.Error())
   }
}

func GetVersion() (v string) {
   getVersionOnce.Do(getVersionJni)
   log.Printf("GetVersion: versionCode: %d, versionName: %s", C.versionCode, C.GoString(C.pVersionName))
   return strings.TrimSpace(C.GoString(C.pVersionName))
}

```

这里有一个技巧，由于`versionName`和`versionCode`几乎是在运行过程不会被修改的，所以使用`sync.Once`来确保`getVersionJni`函数只会被执行了一次，后续的直接从结果里边取出来就可以了。

> 提示：`app.RunOnJVM`如果找不到方法，你可能需要升级一下gomobile。

参考链接：

1. http://tbfungeek.github.io/2016/07/09/Android-进阶之JNI-开发-三访问对象成员变量和成员方法/
2. https://golang.org/cmd/cgo/
3. https://gist.github.com/zchee/b9c99695463d8902cd33
4. https://blog.csdn.net/qq_27278957/article/details/77164353

