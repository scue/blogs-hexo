---
layout: post
title: "React Native之接入微信支付（Android&iOS）"
description: ""
category: 技术
tags: [react native, ios, android]
---

本文章将介绍如何将微信支付App支付功能，接入到React Native应用中，包含Android、iOS，还有JS的调用示例三大模块。

<!-- more -->

# Android

## 一、目录结构


文章所述将需要改动以下文件：

```
app/
└──  build.gradle
app/src/main/
└── AndroidManifest.xml
app/src/main/java/com/xxx/mobile
└── App.kt
app/src/main/java/com/xxx/mobile/wxapi
├── WXPayEntryActivity.kt
├── WxpayModule.kt
└── WxpayPackage.kt
```


## 二、获取AppID
在微信开放平台上`管理中心 / 应用详情`获取得到AppID

![](https://media-1256569450.cos.ap-chengdu.myqcloud.com/blog/15558127222283.jpg)

## 三、获取应用签名（MD5）

具体获取步骤（示范）：
```sh
$ keytool -list -v -keystore ~/.android/demo.keystore | awk '/MD5/{print $2}'
输入密钥库口令:  123456
8B:8A:62:7E:EC:16:00:E4:00:89:13:81:B0:84:C5:D2
$ python -c 'print("8B:8A:62:7E:EC:16:00:E4:00:89:13:81:B0:84:C5:D2".replace(":", "").lower())'
8b8a627eec1600e400891381b084c5d2
```

> PS：需要的是MD5值，小写，而不是别的，之前因为其他同事把这个填写错了，导致每个用户只有微信支付一次，排查了半天...

在微信开放平台上`管理中心 / 修改应用 / 修改开发信息`填入信息：
![](https://media-1256569450.cos.ap-chengdu.myqcloud.com/blog/15558125384432.jpg)

## 三、集成微信支付SDK


```groovy
dependencies {
    implementation "com.tencent.mm.opensdk:wechat-sdk-android-with-mta:+"
    // ... more ...
}
```

> PS: 这样子的集成方式，感觉要比直接导入`.jar`或`.aar`都舒服不少呀。

## 四、创建`wxapi`包名

在`com.xx.mobile`创建包名`wxapi`，注意此处包名一定要为`wxapi`，否则后续将无法处理回调

![](https://media-1256569450.cos.ap-chengdu.myqcloud.com/blog/15558143186727.jpg)

## 五、WxpayModule.kt


```kotlin
package com.xxx.mobile.wxapi

import android.util.Log
import com.facebook.react.bridge.Promise
import com.facebook.react.bridge.ReactApplicationContext
import com.facebook.react.bridge.ReactContextBaseJavaModule
import com.facebook.react.bridge.ReactMethod
import com.facebook.react.bridge.ReadableMap
import com.tencent.mm.opensdk.modelpay.PayReq
import com.tencent.mm.opensdk.openapi.IWXAPI
import com.tencent.mm.opensdk.openapi.WXAPIFactory

internal class WxpayModule(reactContext: ReactApplicationContext) : ReactContextBaseJavaModule(reactContext) {
    companion object {
        val TAG = "WxpayModule"
        var APP_ID = ""
        var promise: Promise? = null
    }

    private val api: IWXAPI = WXAPIFactory.createWXAPI(reactContext, null)

    override fun getName(): String {
        return "Wxpay"
    }

    @ReactMethod
    fun registerApp(APP_ID: String) { // 向微信注册
        WxpayModule.APP_ID = APP_ID
        api.registerApp(APP_ID)
    }

    @ReactMethod
    fun isSupported(promise: Promise) { // 判断是否支持调用微信SDK
        val isSupported = api.isWXAppInstalled
        promise.resolve(isSupported)
    }

    @ReactMethod
    fun pay(order: ReadableMap, promise: Promise) {
        Log.d(TAG, "wxpay pay start")
        WxpayModule.promise = promise
        val request = PayReq()
        request.appId = order.getString("appid") // 应用ID
        request.partnerId = order.getString("partnerid") // 商户号
        request.prepayId = order.getString("prepayid") // 预支付交易会话ID
        request.packageValue = order.getString("package") // 扩展字段（暂定固定值Sign=WXPay）
        request.nonceStr = order.getString("noncestr") // 随机字符串
        request.timeStamp = order.getInt("timestamp").toString() // 时间戳 epoch
        request.sign = order.getString("sign") // 签名，签名方式一定要与统一下单接口使用的一致
        api.sendReq(request)
    }
}
```

## 六、WxpayPackage.kt


```kotlin
package com.xxx.mobile.wxapi

import com.facebook.react.ReactPackage
import com.facebook.react.bridge.NativeModule
import com.facebook.react.bridge.ReactApplicationContext
import com.facebook.react.uimanager.ViewManager

class WxpayPackage : ReactPackage {

    override fun createViewManagers(reactContext: ReactApplicationContext): List<ViewManager<*, *>> {
        return emptyList()
    }

    override fun createNativeModules(reactContext: ReactApplicationContext): List<NativeModule> {
        return listOf(WxpayModule(reactContext))
    }
}
```

## 七、WXPayEntryActivity.kt


```kotlin
package com.xxx.mobile.wxapi

import android.app.Activity
import android.content.Intent
import android.os.Bundle
import android.util.Log
import com.facebook.react.bridge.Arguments
import com.tencent.mm.opensdk.constants.ConstantsAPI
import com.tencent.mm.opensdk.modelbase.BaseReq
import com.tencent.mm.opensdk.modelbase.BaseResp
import com.tencent.mm.opensdk.openapi.IWXAPI
import com.tencent.mm.opensdk.openapi.IWXAPIEventHandler
import com.tencent.mm.opensdk.openapi.WXAPIFactory

class WXPayEntryActivity : Activity(), IWXAPIEventHandler {
    companion object {
        val TAG = "WXPayEntryActivity"
    }

    private var api: IWXAPI? = null

    public override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        api = WXAPIFactory.createWXAPI(this, WxpayModule.APP_ID)
        api?.handleIntent(intent, this)
    }

    override fun onNewIntent(intent: Intent) {
        super.onNewIntent(intent)
        setIntent(intent)
        api?.handleIntent(intent, this)
    }

    override fun onReq(req: BaseReq) {}

    override fun onResp(resp: BaseResp) {
        Log.d(TAG, "wxpay onPayFinish, errCode = " + resp.errCode)
        if (resp.type == ConstantsAPI.COMMAND_PAY_BY_WX) {
            val map = Arguments.createMap()
            map.putInt("errCode", resp.errCode)
            WxpayModule.promise!!.resolve(map)
            finish()
        }
    }
}
```

## 八、App.kt


```kotlin
import com.xxx.mobile.wxapi.WxpayPackage // ← here
class App : Application(), ReactApplication {

    private val mReactNativeHost = object : ReactNativeHost(this) {
        override fun getUseDeveloperSupport(): Boolean {
            return BuildConfig.DEBUG
        }
        override fun getPackages(): List<ReactPackage> {
            return listOf(
                    MainReactPackage(),
                    // ...
                    WxpayPackage() // ← here
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

## 九、AndroidManifest.xml

在AndroidManifest.xml中添加微信支付的activity：

```xml
    <!--微信支付-->
    <activity android:name=".wxapi.WXPayEntryActivity" android:exported="true"/>
```

# iOS

## 一、目录结构

文章所述将涉及以下文件的改动：

```
├── AppDelegate.m
└── Info.plist
Wxapi
├── WXApi.h
├── WXApiObject.h
├── WechatAuthSDK.h
├── WxpayModule.h
└── WxpayModule.m
thirdparty/wechat
└── libWeChatSDK.a
```

## 二、集成微信SDK

SDK下载链接：[WeChatSDK1.8.4.zip](https://res.wx.qq.com/op_res/_bwkvGV3R1XHqZeRgGTd6YHyvEZgBJdv95a2NXVRqHg5sPzhuAghRrQA9bzienfF)

将`WeChatSDK1.8.4.zip`解压到`thirdparty/wechat`，目录结构如下：

```
thirdparty/wechat
└── libWeChatSDK.a
```

`Xcode`→`Project`→`Target`→`General`→`Linked Frameworks and Libraries`，添加以下项：
![](https://media-1256569450.cos.ap-chengdu.myqcloud.com/blog/15558160971201.jpg)


## 三、创建Wxapi的Group

`Xcode` → `Project` → `New Group` → `Wxapi`，并将SDK的头文件拷贝过去，拷贝完成后，目录结构如下：

```
Wxapi/
├── WXApi.h
├── WXApiObject.h
└── WechatAuthSDK.h
```
## 四、WxpayModule.h


```objc
#import <React/RCTBridgeModule.h>
#import <React/RCTLog.h>
#import "WXApiObject.h"
#import "WXApi.h"

@interface WxpayModule : NSObject <RCTBridgeModule, WXApiDelegate>
@end
```

## 五、WxpayModule.m

```objc

#import <Foundation/Foundation.h>
#import "WxpayModule.h"

@implementation WxpayModule
{
  RCTPromiseResolveBlock resolveBlock;
}

- (instancetype)init
{
  self = [super init];
  if (self) {
    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(handleWXPay:) name:@"WXPay" object:nil];
  }
  return self;
}

- (void)dealloc
{
  [[NSNotificationCenter defaultCenter] removeObserver:self];
}

- (void)handleWXPay:(NSNotification *)aNotification
{
  NSString *errCode =  [aNotification userInfo][@"errCode"];
  NSLog(@"wxpay finished, errCode = %@", errCode);
  resolveBlock(@{ @"errCode": errCode });
}

RCT_EXPORT_METHOD(registerApp:(NSString *)APP_ID) {
  NSLog(@"wxpay registerApp, appID %@", APP_ID);
  [WXApi registerApp:APP_ID];   //向微信注册
}

RCT_EXPORT_METHOD(pay:(NSDictionary *)order
                  resolver:(RCTPromiseResolveBlock)resolve
                  rejecter:(RCTPromiseRejectBlock)reject) {
  NSLog(@"wxpay pay start");
  resolveBlock = resolve;
  //调起微信支付
  PayReq *req = [[PayReq alloc] init];
  req.partnerId = [order objectForKey:@"partnerid"];
  req.prepayId = [order objectForKey:@"prepayid"];
  req.nonceStr = [order objectForKey:@"noncestr"];
  req.timeStamp = [[order objectForKey:@"timestamp"] intValue];
  req.package = [order objectForKey:@"package"];
  req.sign = [order objectForKey:@"sign"];
  [WXApi sendReq:req];
}

RCT_REMAP_METHOD(isSupported,   // 判断是否支持调用微信SDK
                 resolver:(RCTPromiseResolveBlock)resolve
                 rejecter:(RCTPromiseRejectBlock)reject) {
  if (![WXApi isWXAppInstalled]) resolve(@NO);
  else resolve(@YES);
}

RCT_EXPORT_MODULE(Wxpay);
@end
```

## 六、AppDelegate.m


```objc
- (BOOL)application:(UIApplication *)application handleOpenURL:(NSURL *)url
{
  return [WXApi handleOpenURL:url delegate:self];
}

- (BOOL)application:(UIApplication *)app openURL:(NSURL *)url options:(NSDictionary *)options
{
  // 微信支付
  if ([url.scheme isEqualToString:@"<YOUR_APPID>"]) {
    return [WXApi handleOpenURL:url delegate:self];
  }
  return NO;
}

//ios9以后的方法
- (BOOL)application:(UIApplication *)application openURL:(NSURL *)url sourceApplication:(NSString *)sourceApplication annotation:(id)annotation
{
  NSLog(@"%s url: %@, sourceApplication: %@", __PRETTY_FUNCTION__, url, sourceApplication);
  // 微信支付 com.tencent.xin
  if ([sourceApplication isEqualToString:@"com.tencent.xin"]) {
    return [WXApi handleOpenURL:url delegate:self];
  }
  return NO;
}

#pragma mark - wx callback
- (void)onReq:(BaseReq *)req
{
  // TODO Something
}

- (void)onResp:(BaseResp *)resp
{
  //判断是否是微信支付回调 (注意是PayResp 而不是PayReq)
  if ([resp isKindOfClass:[PayResp class]]) {
    //发出通知 从微信回调回来之后,发一个通知,让请求支付的页面接收消息,并且展示出来,或者进行一些自定义的展示或者跳转
    NSNotification *notification = [NSNotification notificationWithName:@"WXPay" object:nil userInfo:@{ @"errCode": @(resp.errCode) }];
    [[NSNotificationCenter defaultCenter] postNotification:notification];
  }
}
```

## 七、Info.plist

1) `Xcode`→`Project`→`Target`→`Info`→`URL Types`，填入信息：
- Identifier：`weixin`
- URL Schemes: `YOUR_APPID`

![](https://media-1256569450.cos.ap-chengdu.myqcloud.com/blog/15558164247084.jpg)

2) `Xcode`→`Project`→`Target`→`Info`→`LSApplicationQueriesSchemes`，添加两个item：
![](https://media-1256569450.cos.ap-chengdu.myqcloud.com/blog/15558166731208.jpg)

# JavaScript

1）在React Native的App.js入口处，向微信注册：

```js
  componentWillMount() {
    wxpay.registerApp('<YOUR_APPID>');
  }
```

2）在需要发起微信支付的`pay.js`：

```js
wxpay.pay(params)
  .then(v => {
    __DEV__ && console.log('_wechatPay: result:', v);
  })
  .catch(e => {
    console.warn('_wechatPay: error:', e);
  });
```

参考链接：

1. [一步一步将微信支付集成到 react-native 应用](https://juejin.im/post/5a33253b51882521033449b2)
2. [官网：SDK与DEMO下载](https://pay.weixin.qq.com/wiki/doc/api/app/app.php?chapter=11_1)
3. [官网: 业务流程](https://pay.weixin.qq.com/wiki/doc/api/app/app.php?chapter=8_3)
4. [官网: 统一下单](https://pay.weixin.qq.com/wiki/doc/api/app/app.php?chapter=9_1)
5. [官网: 调起支付接口](https://pay.weixin.qq.com/wiki/doc/api/app/app.php?chapter=9_12&index=2)
6. [微信开放平台apk应用签名获取](https://blog.csdn.net/xzytl60937234/article/details/82941467)


