---
layout: post
title: "React Native之iOS原生模块反向通知JS"
description: ""
category: 技术
tags: [react native, ios]
---

前边介绍了从JS调用iOS原生模块的方法，现在再介绍一下，如何从Native反向通知JS。

# 一、目录结构

```
<PROJECT>-Bridging-Header.h // ← here
ReactEvent
├── ReactEvent.h
├── ReactEvent.m
├── ReactEventHandler.swift
├── ReactEventR.h // ← here
├── ReactEventR.m // ← here
└── ReactEventRHandler.swift // ← here
```
> PS: `ReactEventR`中的`R`表示是反向的意思。

<!-- more -->


# 二、ReactEventR.h

```objc
#import <React/RCTBridgeModule.h>
#import <React/RCTEventEmitter.h>

// ReactEventR: Sending Events to JavaScript
@interface ReactEventR : RCTEventEmitter <RCTBridgeModule>
- (void)emitEvent: (NSString *)event;
@end
```

# 三、ReactEventR.m

```objc
#import "ReactEventR.h"
#import <Foundation/Foundation.h>

@implementation ReactEventR
{
  bool hasListeners;
  bool inited;
}

RCT_EXPORT_MODULE();

- (void)startObserving
{
  printf("ReactEventR: startObserving\n");
  hasListeners = YES;
}

- (void)stopObserving
{
  printf("ReactEventR: stopObserving\n");
  hasListeners = NO;
}

- (NSArray<NSString *> *)supportedEvents
{
  return @[@"ReactEventR"];
}

+ (id)allocWithZone:(NSZone *)zone
{
  static RCTBridge *sharedInstance = nil;
  static dispatch_once_t onceToken;
  dispatch_once(&onceToken, ^{
    sharedInstance = [super allocWithZone:zone];
  });
  return sharedInstance;
}

- (void)waitForListener
{
  if (inited) return;
  for (int i = 0; i < 10; i++) {
    if (hasListeners) break;
    NSLog(@"ReactEventR: wait for listener: %d", i);
    usleep(200*1000);
  }
  inited = true;
}

- (void)emitEvent:(NSString *)event
{
  [self waitForListener];
  if (hasListeners) {
    NSLog(@"ReactEventR: emit event: %@", event);
    [self sendEventWithName:@"ReactEventR" body:event];
  }
}

@end
```

> PS: 注意到`inited`变量了吗，此处的实现并不是很优雅，用于确保JS已有监听者，最多等待两秒时间。

# 四、ReactEventRHandler.swift

```swift
import Foundation
class ReactEventRHandler: NSObject {
  static let shared = ReactEventRHandler()
  var handler: ReactEventR?;
  override init() {
    handler = ReactEventR()
  }
  func emit(event: String) {
    NSLog("ReactEventRHandler: event: %@", event)
    handler?.emitEvent(event)
  }
}
```

# 五、`<PROJECT>-Bridging-Header.h`

由于涉及Objc与Swift混合编程，需要写一个桥接头文件

```objc
#import <React/RCTBridgeModule.h>
#import <React/RCTBridge.h>
#import <React/RCTBundleURLProvider.h>
#import <React/RCTRootView.h>
#import "ReactEventR.h" // ← here
```
然后在Target→Build Settings→Swift Compiler - General→Objective-C Bridging Header，填写上你的`<PROJECT>-Bridging-Header.h`头文件

![](https://media-1256569450.cos.ap-chengdu.myqcloud.com/blog/15557727768229.jpg)

# 六、Swift调用示例
```swift
ReactEventRHandler.shared.emit(event:"SOMETHING") // 单例模式，可直接使用
```

# 七、JS接收使用示例

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