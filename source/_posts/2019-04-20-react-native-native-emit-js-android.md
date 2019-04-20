---
layout: post
title: "React Native之Android原生模块反向通知JS"
description: ""
category: 技术
tags: [react native, android]
---

前边介绍了从JS调用Android原生模块的方法，现在再介绍一下，如何从Native反向通知JS。

# 一、目录结构

```
reactevent
├── ReactEvent.kt
├── ReactEventPkg.kt // ← here
└── ReactEventR.kt   // ← here
```

> PS: `ReactEventR`中的`R`表示是反向的意思。

<!-- more -->

# 二、ReactEventR.kt

```kotlin
package com.xxx.mobile.reactevent

import com.facebook.react.bridge.ReactApplicationContext
import com.facebook.react.bridge.ReactContextBaseJavaModule
import com.facebook.react.modules.core.DeviceEventManagerModule
import java.util.concurrent.CountDownLatch

class ReactEventR(reactContext: ReactApplicationContext) : ReactContextBaseJavaModule(reactContext) {

    companion object {
        var instance: ReactEventR? = null
        private var mLatch = CountDownLatch(1)

        fun init(reactContext: ReactApplicationContext): ReactEventR {
            instance = ReactEventR(reactContext)
            return instance as ReactEventR
        }
    }

    /**
     * Fix bug:
     * Tried to access a JS module
     * before the React instance was fully set up.
     */
    override fun initialize() {
        super.initialize()
        mLatch.countDown()
    }

    fun emit(data: String) {
        try {
            mLatch.await()
            reactApplicationContext
                    .getJSModule(DeviceEventManagerModule.RCTDeviceEventEmitter::class.java)
                    .emit("ReactEventR", data)
        } catch (e: InterruptedException) {
            e.printStackTrace()
        }
    }

    override fun getName(): String {
        return "ReactEventR"
    }
}
```

# 三、ReactEventPkg.kt

```kotlin
package com.xxx.mobile
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
                ReactEvent(context),
                ReactEventR.init(context)) // ← here
    }
}
```

# 四、App.kt

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

# 五、Kotlin发送示例

```kotlin
val emitter: ReactEventR? = ReactEventR.getInstance()
emitter?.emit("SOMETHING")
```
# 六、JS接收使用示例

```js
import {NativeModules,NativeEventEmitter} from 'react-native';
const { ReactEventR } = NativeModules; // ← here
class Home extends React.Component {
    componentDidMount() {
      const ReactEventREmitter = new NativeEventEmitter(ReactEventR); // ← here
      this.rer_subscription = ReactEventREmitter.addListener( // ← here
        'ReactEventR',
        value => {
          // TODO 更新状态
          console.warn("ReactEventR: come value:", value)
        }
      )
      // ...
    }
    componentWillUnmount(){
      // ...
      this.rer_subscription && this.rer_subscription.remove(); // ← here
    }
    // ...
}
```

# 参考链接
1. [iOS开发-与ReactNative交互时bridge is not set](https://www.jianshu.com/p/e071814cfa8d)
2. [kevinejohn/react-native-keyevent#KeyEventModule.java](https://github.com/kevinejohn/react-native-keyevent/blob/master/android/src/main/java/com/github/kevinejohn/keyevent/KeyEventModule.java)
3. [Sending Events to JavaScript#坑](https://facebook.github.io/react-native/docs/native-modules-ios#sending-events-to-javascript)
4. [Sending events to JavaScript from your native module in React Native](https://blog.callstack.io/sending-events-to-javascript-from-your-native-module-in-react-native-29244f890e04)