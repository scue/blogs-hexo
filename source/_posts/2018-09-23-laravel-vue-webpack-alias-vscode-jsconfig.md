---
layout: post
title: "Laravel配置Webpack的alias以及VScode配置jsconfig.js"
description: ""
category: 技术
tags: [laravel, webpack, alias, jsconfig]
---

现阶段一些前端界面还是使用了Laravel框架+Vue结合起来写，每次`import`的时候，都是长长的相对路径，比如 `../../../utils/utils`，看起来很别扭。

花上两分钟时间，让你在VSCode可以使用`import utils from "@/utils/utils"`的`import`语句; 并且，还可以让VScode继续保持着代码自动补全功能。

<!-- more -->

# Alias配置

Lavarel的Webpack配置文件是`webpack.mix.js`，往里边添加内容：

```js
mix.webpackConfig({
  resolve: {
    alias: {
      "@": path.resolve(__dirname, "resources/assets/js")
    }
  }
})
```

修改之后的内容如下：


```js
let mix = require("laravel-mix")

mix.webpackConfig({
  resolve: {
    alias: {
      "@": path.resolve(__dirname, "resources/assets/js")
    }
  }
})

mix
  .js("resources/assets/js/app.js", "public/js")
  .sass("resources/assets/sass/app.scss", "public/css")

```

> 即将 `@` 解析至 `resources/assets/js` 目录

# VScode配置

这样子解析之后，VScode就不能自动补全代码了，顺便我们一起配置一下，创建一个文件`jsconfig.js`（与`webpack.mix.js`同级）

内容如下：

```js
{
  "compilerOptions": {
    "baseUrl": ".",
    "include": ["resources/assets/js/**/*"],
    "paths": {
      "@/*": ["resources/assets/js/*"]
    }
  },
  "exclude": [
    "app",
    "bootstrap",
    "config",
    "database",
    "node_modules",
    "public",
    "routes",
    "storage",
    "tests",
    "vendor"
  ]
}
```

OK，这样子VScode的代码补全也都好了，可以爽快的编码了。~