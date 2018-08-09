---
layout: post
title: "govendor实用技巧"
description: ""
category: 技术
tags: [go govendor]
---

# 关于`govendor`

govendor是go语言依赖管理工具，推荐使用 https://github.com/kardianos/govendor 这个版本。

```sh
go get -u -v github.com/kardianos/govendor
```

第一次初始化的时候，只需要`govendor init`命令行执行一下就可以了。

> 它的使用起来就像nodejs的`yarn`或`npm`包管理工具一样简单！

<!-- more -->

# 当我们需要下载一个依赖包的时候

1. 尽量使用`govendor fetch -v` 来安装依赖的包，使用这个方法，不但可以下载自身的包，还可以下载依赖
2. 若使用`govendor get -v`，与官网所述`Like "go get" but copies dependencies into a "vendor" folder`，实际上只复制了依赖的包进去而已..
3. `govendor add -v`也不好使，`Add packages from $GOPATH`只会从本地加载依赖的包

综上，应当这样子来下载依赖包：

```sh
govendor fetch -v github.com/gin-gonic/gin/...@v1.2
```

# 当我们需要依赖包目录下所有模块的时候

```sh
govendor fetch -v github.com/gin-gonic/gin@v1.2 # 只拷贝gin/目录的内容，而不包含其子目录
govendor fetch -v github.com/gin-gonic/gin/...@v1.2 # 可以得到gin/目录，及其所有子目录
```

> `@v1.2` 表示使用 `v1.2` 这个版本，其实是对应的是 `git tag` 为 `v1.2` 的revision，这个功能也是很实用！

# 当我们所需要的依赖包不是官方的时候

有时候我们发现了第三方依赖包存在一些问题，此时我们修复了之后，期望使用自己的仓库的时候，可以这样子

```sh
govendor get -v 'github.com/go-sql-driver/mysql::github.com/scue/go-mysql'
```

> 原仓库的`github.com/go-sql-driver/mysql`存在一个小问题，此时期望使用自己修复过的`github.com/scue/go-mysql`

这样子的方式，真的很好用，也很实用！

# 不要将整个`vendor/`目录的内容都提到git仓库

我们只需要`vendor/vendor.json`这个一文件，其他都不需要！
我们只需要`vendor/vendor.json`这个一文件，其他都不需要！
我们只需要`vendor/vendor.json`这个一文件，其他都不需要！

重要的话说三遍。

当我们需要将代码同步的时候，只需要：

```sh
govendor sync -v
```

> 它就像 `yarn install` 一样简单！

它是有缓存的，同步效率很高。

同理，你的`.gitignore`文件应当长成这样子的：

```
# Created by https://www.gitignore.io/api/go
### Go ###
# Binaries for programs and plugins
*.exe
*.exe~
*.dll
*.so
*.dylib
# Test binary, build with `go test -c`
*.test
# Output of the go coverage tool, specifically when used with LiteIDE
*.out

### Go Patch ###
/vendor/
/Godeps/
# End of https://www.gitignore.io/api/go

# COSTOM
/.tmp
!/vendor/vendor.json
```

# 对应关系

若你有`nodejs`的一些经验，这里列出一些关系供您参考：

| 命令说明 | yarn | govendor |
| --- | --- | --- |
| 初始化 | yarn init | govendor init |
| 添加依赖（包含依赖的依赖） | yarn add | govendor fetch |
| 删除依赖 | yarn remove | govendor remove |
| 同步依赖 | yarn install | govendor sync |

> 提示：你可能需要经常使用 `govendor list` 来查看是否有一些依赖已经安装到了 `$GOPATH` 的全局环境了，如果有，并且是必须的，建议通过 `govendor fetch` 将它们添加进来。