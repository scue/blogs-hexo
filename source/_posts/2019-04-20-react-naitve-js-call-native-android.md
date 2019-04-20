---
layout: post
title: "React Native之JS调用Android原生模块"
description: ""
category: 技术
tags: [react native, android]
---

React Native开发免不了需要从JS调用原生的模块，这里演示了如何从JS调用原生模块，并返回数据的方法。

# 一、目录结构

```
reactevent
├── ReactEvent.kt
└── ReactEventPkg.kt
```

<!-- more -->

# 二、创建reactevent的package

在项目中创建一个名为 `reactevent` 的包名
![](https://media-1256569450.cos.ap-chengdu.myqcloud.com/blog/15557704734321.jpg)

# 三、ReactEvent.kt

```kotlin
package com.xxx.mobile.reactevent

import android.util.Log
import com.alibaba.fastjson.JSON
import com.facebook.react.bridge.Promise
import com.facebook.react.bridge.ReactApplicationContext
import com.facebook.react.bridge.ReactContextBaseJavaModule
import com.facebook.react.bridge.ReactMethod
import com.xxx.mobile.CtrlEvent
import org.greenrobot.eventbus.EventBus

class ReactEvent(context: ReactApplicationContext) : ReactContextBaseJavaModule(context) {

    val TAG = "ReactEvent"
    override fun getName(): String {
        return "ReactEvent"
    }

    /**
     * let event = {
     *      'func': 'start',// String
     *      'args': []      // JsonArray
     *  }
     */
    @ReactMethod
    fun send(event: String, promise: Promise) {
        val e = JSON.parseObject(event)
        val func = e.getString("func")
        Log.i(TAG, "ReactEvent: send: func: $func")
        when (func) {
            "start" -> handleStart(promise)
            "stop" -> handleStop(promise)
            "status" -> handleStatus(promise)
            "set_profile" -> handleSetProfile(e, promise)
            "native_view" -> handleNativeView(promise)
            "get_channel" -> handleGetChannel(promise)
            else -> {
                promise.reject(TAG, "未知的函数: $func")
            }
        }
    }

    // 启动，需要跑到MainActivity上跑
    private fun handleStart(promise: Promise) {
        EventBus.getDefault().post(CtrlEvent("start"))
        promise.resolve("执行成功，请使用status事件检测运行状态")
    }

    // 停止，需要跑到MainActivity上跑
    private fun handleStop(promise: Promise) {
        EventBus.getDefault().post(CtrlEvent("stop"))
        promise.resolve("执行成功，请使用status事件检测运行状态")
    }

    // ... more ...
}

```

# 四、ReactEventPkg.kt

```kotlin
package com.xxx.mobile.reactevent

import android.view.View
import com.facebook.react.bridge.NativeModule
import com.facebook.react.bridge.ReactApplicationContext
import java.util.Collections.emptyList
import com.facebook.react.ReactPackage
import com.facebook.react.uimanager.ReactShadowNode
import com.facebook.react.uimanager.ViewManager


class ReactEventPkg : ReactPackage {
    /**
     * @param context 上下文
     * @return 需要调用的原生控件
     */
    override fun createViewManagers(context: ReactApplicationContext?):
            MutableList<ViewManager<View, ReactShadowNode<*>>> {
        return emptyList()
    }

    /**
     * @param context 上下文
     * @return 需要调用的原生模块
     */
    override fun createNativeModules(context: ReactApplicationContext): List<NativeModule> {
        return listOf<NativeModule>(
                ReactEvent(context)) // ← here
    }
}
```

# 五、App.kt

```kotlin
class App : Application(), ReactApplication {

    private val mReactNativeHost = object : ReactNativeHost(this) {
        override fun getUseDeveloperSupport(): Boolean {
            return BuildConfig.DEBUG
        }
        override fun getPackages(): List<ReactPackage> {
            return Arrays.asList<ReactPackage>(
                    MainReactPackage(),
                    // ...
                    ReactEventPkg() // ← here
            )
        }
        override fun getJSMainModuleName(): String {
            return "index"
        }
    }
    // ...
}
```

> PS: 此`App.kt`实际是由`MainApplication.java`转为Kotlin语言的文件
