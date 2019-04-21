---
layout: post
title: "React Native之iOS远程真机调试"
description: ""
category: 技术
tags: [react native, ios]
---

本文章描述如何在真机环境上远程调试js代码，有一些开发工作（如支付测试）是需要在真机环境上才具备测试条件的，本文所述的方法，对于这类环境来说，非常有帮助。


<!-- more -->



方法其实很简单.

# 一、AppDelegate.m

只需要在`AppDelegate.m`文件中，把`jsCodeLocation`改为以下配置：

```objc

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
NSURL *jsCodeLocation;
  #ifdef DEBUG // 动态加载
  #if (TARGET_OS_SIMULATOR) // 模拟器
  jsCodeLocation = [[RCTBundleURLProvider sharedSettings] jsBundleURLForBundleRoot:@"index" fallbackResource:nil];
  #else // 真机测试
  jsCodeLocation = [NSURL URLWithString:@"http://<REMOTE_HOST>:8081/index.bundle?platform=ios&dev=true&minify=false"];
  #endif
  #else
  // 静态加载
  jsCodeLocation = [[NSBundle mainBundle] URLForResource:@"main" withExtension:@"jsbundle"];
  #endif
  // ... more ...
}
```

如果以DEBUG进行编译时：
- 在模拟器上运行时，还是走原来的方式
- 在真机环境运行时，从`<REMOTE_HOST>:8081`这个地址上加载js

如果以非DEBUG进行编译时：
- 还是默认走回从`main.jsbundle`文件加载的方式

其中，`<REMOTE_HOST>:8081`一定是要从手机上可以访问的地址。

# 二、配置允许从http不安全访问

由于iOS安全限制，默认不允许使用http（而必须使用https），由于我们这是调试，有需要的可针对性的放开。

参考以下配置进行配置：

```plist
    <key>NSAppTransportSecurity</key>
    <dict>
        <key>NSAllowsArbitraryLoads</key>
        <true/>
        <key>NSExceptionDomains</key>
        <dict>
            <key>link****.com</key>
            <dict>
                <key>NSExceptionAllowsInsecureHTTPLoads</key>
                <true/>
            </dict>
            <key>localhost</key>
            <dict>
                <key>NSExceptionAllowsInsecureHTTPLoads</key>
                <true/>
            </dict>
        </dict>
    </dict>
```

![](https://media-1256569450.cos.ap-chengdu.myqcloud.com/blog/15558277887776.jpg)



---

# 三、**扩展阅读**

使用frp来将本地的端口映射至公网服务器的端口上。

由于在公司网络环境的限制，WiFi环境的手机不能访问得到内网环境暴露的端口，因此，需要把内网的端口映射到公网环境。

简单的配置方法：


1）公网服务器配置
```sh
wget https://github.com/fatedier/frp/releases/download/v0.26.0/frp_0.26.0_linux_amd64.tar.gz
tar xvf frp_0.26.0_linux_amd64.tar.gz
cd frp_0.26.0_linux_amd64
setsid ./frps -c ./frps.ini >frps.log 2>&1
```

2）内网客户端配置

```sh

# 前提条件：下载frp，并解压至  ~/Downloads/frp_0.16.0_darwin_amd64
cd ~/Downloads/frp_0.16.0_darwin_amd64

cat <<EOF > frpc-rn-me.ini
[common]
server_addr = link****.com
server_port = 7000
# auth token
token = ******** // 访问TOKEN，不要随意告诉别人哦~

[rn-8081]
type = tcp
local_ip = 127.0.0.1
local_port = 8081
remote_port = 8081
EOF
./frpc -c ./frpc-rn-me.ini
```

这样子运行起来之后，就可以随时随地地对iOS真机远程调试了（JS）。