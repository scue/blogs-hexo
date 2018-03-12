---
layout: post
date: 2018-02-08 00:00:00
title: "iOS获取应用程序信息"
description: ""
category: 技术
tags: 
  - iOS
---

主要获取了APP应用程序名、以及版本号。

```swift
let infoDict = Bundle.main.infoDictionary!;
let appDisplayName = infoDict["CFBundleDisplayName"];
let appShortVersion = infoDict["CFBundleShortVersionString"];
let appBuildVersion = infoDict["CFBundleVersion"];
let resolveDict = [
    "appDisplayName": appDisplayName,
    "appShortVersion": appShortVersion,
    "appBuildVersion": appBuildVersion,
]
do {
    let json = try JSONSerialization.data(withJSONObject: resolveDict, options: []);
    let jsonStr = String(data: json, encoding: .utf8);
    resovler(jsonStr);
} catch {
    rejecter("Get AppInfo error", error.localizedDescription, nil);
}
```

