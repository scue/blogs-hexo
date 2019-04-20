---
layout: post
title: "React Native之JS调用iOS原生模块"
description: ""
category: 技术
tags: [react native, ios]
---

React Native开发免不了需要从JS调用原生的模块，这里演示了如何从JS调用原生模块，并返回数据的方法。

# 一、目录结构
```
<PROJECT>-Bridging-Header.h
ReactEvent
├── ReactEvent.h
├── ReactEvent.m
└── ReactEventHandler.swift
```

<!-- more -->


# 二、创建ReactEvent的Group

在项目的主目录下，New Group，命名为`ReactEvent`

![](https://media-1256569450.cos.ap-chengdu.myqcloud.com/blog/%E5%9B%BE%E7%89%87.png)

# 三、ReactEvent.h

在ReactEvent的Group下新建头文件`ReactEvent.h`

```c
#import <React/RCTBridgeModule.h>
#import <React/RCTBridge.h>
@interface ReactEvent: NSObject<RCTBridgeModule>
@end
```

# 四、ReactEvent.m

在ReactEvent的Group下新建源文件`ReactEvent.m`

```objc
#import "ReactEvent.h"

// Swift混编：<Project>-swift.h
#import <PROJECT-Swift.h>

@implementation ReactEvent

RCT_EXPORT_MODULE();

// 对应JS调用：NativeModules.ReactEvent.send(event: String).then().catch()
RCT_EXPORT_METHOD(send:(NSString *)event resolver:(RCTPromiseResolveBlock)resolve rejecter:(RCTPromiseRejectBlock)reject){
  [ReactEventHandler.sharedInstance handleWithEvent:event Resolve:resolve Reject:reject];
}
@end
```

# 五、ReactEventHandler.swift

在ReactEvent的Group下新建源文件`ReactEventHandler.swift`

```swift
import Foundation

@objc(ReactEventHandler)
class ReactEventHandler: NSObject {
  static let shared = ReactEventHandler()

  @objc open class func sharedInstance() -> ReactEventHandler {
    return ReactEventHandler.shared
  }

  @objc open func handle(Event event: String,
                         Resolve resolve: @escaping RCTPromiseResolveBlock,
                         Reject reject: @escaping RCTPromiseRejectBlock) {
    NSLog("ReactEventHandler: handle event: \(event)")
    let eventData = event.data(using: String.Encoding.utf8)
    do {
      // JSON 解析
      let jsonObj = try JSONSerialization.jsonObject(with: eventData!, options: [JSONSerialization.ReadingOptions.mutableLeaves])

      guard
        let json = jsonObj as? NSDictionary,
        let action = json["func"] as? String
      else {
        reject("ReactEventHandler", "传入的JSON解析异常，请检查: \(event).", nil)
        return
      }

      switch action {
      case "start":
        NSLog("%@: start ...", #function)
        resolve("OK")
      case "stop":
        NSLog("%@: stop ...", #function)
        resolve("OK")
      case "status":
        NSLog("%@: status ...", #function)
        resolve("OK")
      case "set_profile":
        NSLog("%@: set_profile ...", #function)
        resolve("OK")
      default:
        reject("ReactEventHandler", "不支持的指令\(action)!", nil)
      }
    } catch {
      reject("ReactEventHandler", error.localizedDescription, nil)
    }
  }
}
```

# 六、`<PROJECT>-Bridging-Header.h`

由于涉及Objc与Swift混合编程，需要写一个桥接头文件

```objc
#import <React/RCTBridgeModule.h>
#import <React/RCTBridge.h>
#import <React/RCTBundleURLProvider.h>
#import <React/RCTRootView.h>
```

然后在Target→Build Settings→Swift Compiler - General→Objective-C Bridging Header，填写上你的`<PROJECT>-Bridging-Header.h`头文件

![](https://media-1256569450.cos.ap-chengdu.myqcloud.com/blog/15557727768229.jpg)


# 七、JS调用示例

```js
let event = {
  func: 'start'
};
NativeModules.ReactEvent.send(JSON.stringify(event))
  .then(console.log)
  .catch(console.warn);
```

参考链接：https://facebook.github.io/react-native/docs/native-modules-ios