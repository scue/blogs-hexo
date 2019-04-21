---
layout: post
title: "React Native之接入支付宝（Android&iOS）"
description: ""
category: 技术
tags: [react native, ios, android]
---

本文章将介绍如何将支付宝App支付功能，接入到React Native应用中，包含Android、iOS，还有JS的调用示例三大模块。


<!-- more -->


# Android

## 一、目录结构

文章所述将需要改动以下文件：

```
app/
├── build.gradle
└── proguard-rules.pro
app/libs/
└── alipaySdk-20180601.jar
app/src/main/
└── AndroidManifest.xml
app/src/main/java/com/xxx/mobile
└── App.kt
app/src/main/java/com/xxx/mobile/alipay
├── AlipayModule.kt
└── AlipayPackage.kt
```

## 二、集成SDK

SDK下载链接：[iOS&Android版资源 v15.5.5](https://openhome.alipay.com/doc/sdkResPackageDownLoad.resource?code=983f8f41979543b697040d9c01fd3dae)

> PS: 支付宝官方在2019-04-18刚刚更新了文档，提供了aar的SDK集成方式，看样子集成起来会更方便一些，目前我还没进行验证，因此这里的SDK下载链接其实是上一个版本的、已验证过的SDK。

**集成步骤：**

1. 将SDK的文件`alipaySdk-20180601.jar`拷贝至`app/libs`目录下
2. 在`build.gradle`中，配置`dependencies`

    ```groovy
    dependencies {
        implementation fileTree(dir: "libs", include: ["*.jar"])
        // ... more ...
    }
    ```
    
## 三、AndroidManifest.xml

在`AndroidManifest.xml`中添加支付宝SDK运行所需的权限和`Activities`:

```xml
    <!-- 支付宝所需要的权限 -->
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
    <uses-permission android:name="android.permission.READ_PHONE_STATE" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    
    <!-- 支付宝 activities -->
    <activity
        android:name="com.alipay.sdk.app.H5PayActivity"
        android:configChanges="orientation|keyboardHidden|navigation|screenSize"
        android:exported="false"
        android:screenOrientation="behind"
        android:windowSoftInputMode="adjustResize|stateHidden" >
    </activity>

    <activity
        android:name="com.alipay.sdk.app.H5AuthActivity"
        android:configChanges="orientation|keyboardHidden|navigation"
        android:exported="false"
        android:screenOrientation="behind"
        android:windowSoftInputMode="adjustResize|stateHidden" >
    </activity>
```

## 四、新建alipay的package

在工程中创建一个package，如`com.xxx.mobile.alipay`

```
app/src/main/java/com/xxx/mobile/alipay
├── AlipayModule.kt
└── AlipayPackage.kt
```

## 五、AlipayModule.kt

```kotlin
package com.xxx.mobile.alipay

import android.util.Log
import com.alipay.sdk.app.PayTask
import com.facebook.react.bridge.*

class AlipayModule(reactContext: ReactApplicationContext) : ReactContextBaseJavaModule(reactContext) {
    companion object {
        val TAG = "AlipayModule"
    }

    override fun getName(): String {
        return "Alipay"
    }

    @ReactMethod
    fun pay(orderInfo: String, promise: Promise) {
        Thread(Runnable {
            Log.d(TAG, "alipay pay start")
            val map = Arguments.createMap()
            val alipay = PayTask(currentActivity)
            val result = alipay.payV2(orderInfo, true)
            for ((key, value) in result) {
                map.putString(key, value)
            }
            Log.d(TAG, "alipay pay finish$map")
            promise.resolve(map)
        }).start()
    }
}
```

## 六、AlipayPackage.kt

```kotlin
package com.xxx.mobile.alipay

import com.facebook.react.ReactPackage
import com.facebook.react.bridge.NativeModule
import com.facebook.react.bridge.ReactApplicationContext
import com.facebook.react.uimanager.ViewManager

class AlipayPackage : ReactPackage {

    override fun createViewManagers(reactContext: ReactApplicationContext): List<ViewManager<*, *>> {
        return emptyList()
    }

    override fun createNativeModules(reactContext: ReactApplicationContext): List<NativeModule> {
        return listOf<NativeModule>(AlipayModule(reactContext))
    }
}
```

## 七、App.kt


```kotlin
import com.xxx.mobile.alipay.AlipayPackage // ← here
class App : Application(), ReactApplication {

    private val mReactNativeHost = object : ReactNativeHost(this) {
        override fun getUseDeveloperSupport(): Boolean {
            return BuildConfig.DEBUG
        }
        override fun getPackages(): List<ReactPackage> {
            return listOf(
                    MainReactPackage(),
                    // ...
                    AlipayPackage() // ← here
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

## 八、proguard-rules.pro

在`app/proguard-rules.pro`文件中，添加以下内容：

```
# 支付宝
-keep class com.alipay.android.app.IAlixPay{*;}
-keep class com.alipay.android.app.IAlixPay$Stub{*;}
-keep class com.alipay.android.app.IRemoteServiceCallback{*;}
-keep class com.alipay.android.app.IRemoteServiceCallback$Stub{*;}
-keep class com.alipay.sdk.app.PayTask{ public *;}
-keep class com.alipay.sdk.app.AuthTask{ public *;}
-keep class com.alipay.sdk.app.H5PayCallback {
    <fields>;
    <methods>;
}
-keep class com.alipay.android.phone.mrpc.core.** { *; }
-keep class com.alipay.apmobilesecuritysdk.** { *; }
-keep class com.alipay.mobile.framework.service.annotation.** { *; }
-keep class com.alipay.mobilesecuritysdk.face.** { *; }
-keep class com.alipay.tscenter.biz.rpc.** { *; }
-keep class org.json.alipay.** { *; }
-keep class com.alipay.tscenter.** { *; }
-keep class com.ta.utdid2.** { *;}

```

# iOS

## 一、目录结构

文章所述将涉及以下文件的改动：

```
├── AppDelegate.m
└── Info.plist
Alipay/
├── AlipayModule.h
└── AlipayModule.m
thirdparty/alipay/
├── AlipaySDK.bundle
└── AlipaySDK.framework
```

## 二、集成SDK

1) 将下载的SDK文件，拷贝到`ios/thirdparty/alipay/`目录下，如图所示：
![](https://media-1256569450.cos.ap-chengdu.myqcloud.com/blog/15558083103362.jpg)
2) 然后通过`Xcode`→`Project`→`Add Files to "<Project>" ...`，把`AlipaySDK.bundle`和`AlipaySDK.framework`添加至项目中，如下图：
![](https://media-1256569450.cos.ap-chengdu.myqcloud.com/blog/15558085172719.jpg)
3) `Xcode`→`Project`→`Target`→`General`→`Linked Frameworks and Libraries`，添加以下项
![](http://img01.taobaocdn.com/top/i1/LB1PlBHKpXXXXXoXXXXXXXXXXXX#align=left&display=inline&height=361&originHeight=538&originWidth=1112&status=done&width=746)

## 三、AlipayModule.h


```objc
#import <React/RCTBridgeModule.h>
#import <React/RCTLog.h>

@interface AlipayModule : NSObject <RCTBridgeModule>
@end
```

## 四、AlipayModule.m

```objc
#import <Foundation/Foundation.h>
#import "AlipayModule.h"
#import <AlipaySDK/AlipaySDK.h>

@implementation AlipayModule
{
  RCTPromiseResolveBlock resolveBlock;
}

- (instancetype)init
{
  self = [super init];
  if (self) {
    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(handleAliPay:) name:@"AliPay" object:nil];
  }
  return self;
}

- (void)handleAliPay:(NSNotification *)notification
{
  NSDictionary *resultDic =  [notification userInfo];
  NSLog(@"alipay pay finish: result: %@", resultDic);
  resolveBlock(resultDic);
}

RCT_EXPORT_METHOD(pay:(NSString *)orderInfo
                  resolver:(RCTPromiseResolveBlock)resolve
                  rejecter:(RCTPromiseRejectBlock)reject) {
  resolveBlock = resolve;
  NSLog(@"alipay pay start");
  NSString *appScheme = @"alipay<appname>"; // 注意，此处应填写的是Info.plist文件中的URL Schemes！
  // 调用支付结果开始支付
  [[AlipaySDK defaultService] payOrder:orderInfo fromScheme:appScheme callback:^(NSDictionary *resultDic) {
    NSLog(@"alipay pay finish: result: %@", resultDic);
  }];
}

RCT_EXPORT_MODULE(Alipay);
@end
```

> PS: 别指望`AlipayModule.m`中的`callback`会真正的执行，实际上支付结果是通过`AppDelegate.m`的`(BOOL)application:(UIApplication *)app openURL:(NSURL *)url options:(NSDictionary *)options`来获得的。`AppDelegate.m`收到支付结果之后，通过发送通知的形式来达到这个文件中的`(void)handleAliPay:(NSNotification *)notification`函数，再回调给JS前端。

## 五、AppDelegate.m


```objc
#import <AlipaySDK/AlipaySDK.h>

- (BOOL)application:(UIApplication *)app openURL:(NSURL *)url options:(NSDictionary *)options
{
  NSLog(@"%s url: %@", __PRETTY_FUNCTION__, url);
  // 支付宝
  if ([url.host isEqualToString:@"safepay"]) {
    [self handleAlipayResult:url];
    return YES;
  }
  return NO;
}

- (BOOL)application:(UIApplication *)application openURL:(NSURL *)url sourceApplication:(NSString *)sourceApplication annotation:(id)annotation
{
  NSLog(@"%s url: %@, sourceApplication: %@", __PRETTY_FUNCTION__, url, sourceApplication);
  // 支付宝 com.alipay.iphoneclient
  if ([url.host isEqualToString:@"safepay"]) {
    [self handleAlipayResult:url];
    return YES;
  }
  return NO;
}

// 处理支付宝支付结果
- (void)handleAlipayResult:(NSURL *)url
{
  // 支付跳转支付宝钱包进行支付，处理支付结果
  [[AlipaySDK defaultService] processOrderWithPaymentResult:url standbyCallback:^(NSDictionary *resultDic) {
    NSLog(@"alipy result = %@", resultDic);
    [self postAlipayNotify:resultDic];
  }];

  // 授权跳转支付宝钱包进行支付，处理支付结果
  [[AlipaySDK defaultService] processAuth_V2Result:url standbyCallback:^(NSDictionary *resultDic) {
    NSLog(@"alipay result via auth = %@", resultDic);
    [self postAlipayNotify:resultDic];
  }];
}

// 发送支付宝结果通知
- (void)postAlipayNotify:(NSDictionary *)resultDic
{
  NSNotification *notification = [NSNotification notificationWithName:@"AliPay" object:nil userInfo:resultDic];
  [[NSNotificationCenter defaultCenter] postNotification:notification];
}
```

## 六、Info.plist

`Xcode`→`Project`→`Target`→`Info`→`URL Types`，添加`alipay<appname>`

![](https://media-1256569450.cos.ap-chengdu.myqcloud.com/blog/15558095272927.jpg)

> PS: URL Schemes填写的内容，建议跟商户App有一定的标识度，要做到和其他的商户App不重复，否则可能会导致支付宝返回的结果无法正确跳回商户App。提示来源：[官网: iOS集成流程](https://docs.open.alipay.com/204/105295/)。

# JavaScript


```js
_aliPay = () => {
  this.setState({ payType: 1 });
  let params = {
    sid: this.props.sid,
    good_id: this.state.selectedid,
    pay_type: PayType.ALIPAY
  };

  this.props.purchase(
    params, // 第一步：去后台下单
    async result => {
      if (!TextUtil.isEmpty(result.data)) { // 第二步：下单成功后，返回调用支付宝App所需的参数信息
        let orderId = result.data.order_id;
        let payRes = await alipay.pay(result.data.pay_info); // 第三步：调用支付宝App支付
        console.log('payresult: ', payRes);
        if (!TextUtil.isEmpty(payRes)) {
          let payStatus = parseInt(payRes.resultStatus);
          switch (payStatus) {
            case 9000:
              this._queryPurchaseStatus(orderId); // 第四步：去后台查询订单状态、发货状态
              ToastUtil.showToast('订单支付成功');
              break;
            case 8000:
              this._queryPurchaseStatus(orderId);
              ToastUtil.showToast('正在处理中，支付结果未知');
              break;
            case 5000:
              ToastUtil.showToast('重复请求');
              break;
            case 4000:
              ToastUtil.showToast('订单支付失败');
              break;
            case 6001:
              ToastUtil.showToast('用户中途取消');
              break;
            case 6002:
              ToastUtil.showToast('网络连接出错');
              break;
            case 6004:
              ToastUtil.showToast('支付结果未知');
              break;
            default:
              ToastUtil.showToast('支付异常');
              break;
          }
        }
      }
    },
    err => {
      ToastUtil.showToast('支付失败！'); // TODO 判断失败类型
      console.warn('_aliPay: 支付失败，错误信息：', err);
    }
  );
};
```

参考链接：

1. [一步一步将支付宝集成到 react-native 应用](https://juejin.im/post/5a2663946fb9a04517050de6)
2. [官网：客户端 Android 集成流程](https://docs.open.alipay.com/204/105296/)
3. [官网：App支付iOS集成流程](https://docs.open.alipay.com/204/105295/)
4. [官网: resultStatus结果码含义](https://docs.open.alipay.com/204/105301/)