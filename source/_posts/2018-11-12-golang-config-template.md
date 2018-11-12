---
layout: post
title: "GO语言一个配置文件加载、重载模板"
description: ""
category: 技术
tags: [go]
---

也许你也会有我一样的困扰，开发了很多的GO语言程序，发现很多程序都需要一个配置文件，这有一个简单的配置文件解析、加载、重载模板...

<!-- more -->

```go
package config
import "sync"
var (
    configFile string
    cachedConfig Config
    cachedConfigMux sync.RWMutex
)
type Config struct {
    // TODO add your configs
}
func ParseConfig(file string) (cfg Config, e error) {
    // TODO parse your configs
    return
}
// 加载配置
func LoadConfig(file string) (e error) {
    cfg, e := ParseConfig(file)
    if e != nil {
        return
    }
    cachedConfig = cfg
    configFile = file
    return
}
// 重新加载配置
func ReloadConfig() (e error) {
    cfg, e := ParseConfig(configFile)
    if e != nil {
        return
    }
    cachedConfigMux.Lock()
    defer cachedConfigMux.Unlock()
    cachedConfig = cfg
    return
}
// 获取配置
func Get() *Config {
    cachedConfigMux.RLock()
    defer cachedConfigMux.RUnlock()
    return &cachedConfig
}
```

> 提示：通常你的配置文件格式可以是json, yaml或者toml，json文件没有注释的功能，不是很建议使用，个人比较喜欢使用yaml，看起来很简洁。