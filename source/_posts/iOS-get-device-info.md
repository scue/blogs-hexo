---
layout: post
date: 2018-02-08 00:00:00
title: "获取iOS设备信息"
description: ""
category: 技术
tags: 
  - iOS
---




这是我做一个iOS版本运维App的时候写的代码，记录一下：

```swift
let resolveDict = [
    "systemName": UIDevice.current.systemName,
    "systemVersion": UIDevice.current.systemVersion,
    "identifierForVendor": UIDevice.current.identifierForVendor?.description ?? "",
    "model": UIDevice.current.model,
    "localizedModel": UIDevice.current.localizedModel
    ]
do {
    let json = try JSONSerialization.data(withJSONObject: resolveDict, options: []);
    let jsonStr = String(data: json, encoding: .utf8);
    resovler(jsonStr);
} catch {
    rejecter("Get AppInfo error", error.localizedDescription, nil);
}

```

