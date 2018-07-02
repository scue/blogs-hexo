---
layout: post
title: "一种在Lavarel-Admin上使用Vue组件的方式"
description: ""
category: 技术
tags: [php, laravel, pjax, vue]
---

Laravel-Admin框架总体还是很不错的，构建一个CMDB可以很快的完成。

同时这个框架的局限性也比较明显，比如我想做<font color=#f58220>一些漂亮一点的UI和友好的交互界面，就不得不考使用Vue来开发一些组件</font>了。

本文章主要讲述：

- 如何在`Laravel-admin`上使用`Vue`进行前端界面的开发
- 如何解决`Pjax`和`Vue`组件不能友好的共存的问题

# 背景知识

如何下的两个按钮「**上传升级包**」和「**批量SN导入**」都是使用Vue进行开发的。

因为Vue直的很方便，使用`Vue`+`ElementUI`可以快速的开发一个相对好看又简约的前端界面。

文件目录结构：

```txt
resources
├── assets/
│   ├── js/
│   │   ├── components/
│   │   │   ├── hide_product_info/
│   │   │   │   ├── OtaBulkSn.vue        // → 批量SN导入
│   │   │   │   ├── OtaBulkSnAppend.vue
│   │   │   │   ├── OtaBulkSnCreate.vue
│   │   │   │   ├── OtaBulkSnDestroy.vue
│   │   │   │   ├── OtaBulkSnRemove.vue
│   │   │   │   ├── OtaBulkSnView.vue
│   │   │   │   └── OtaPkgUpload.vue     // → 上传OTA升级包
│   │   │   └── ExampleComponent.vue
│   │   ├── app.js
│   │   └── bootstrap.js
...
```

其中的`OtaBulkSn.vue`是批量SN导入，`OtaPkgUpload.vue`是上传OTA升级包

它俩都是单文件的组件，在`app.js`是这样子加载进来的：

```js
Vue.component(
  "ota-bulksn-component",
  require("./components/onecloudpro/OtaBulkSn.vue")
)
Vue.component(
  "otapkg-upload-component",
  require("./components/onecloudpro/OtaPkgUpload.vue")
)
```

在App.js里边目前只是「全局注册」了这两个组件`ota-bulksn-component`和`otapkg-upload-component`

实际上加载了App.js并没有直接的使用到这两个文件，后边我会提到何时去使用这两个组件

## 上传OTA升级包

![](https://media-1256569450.cos.ap-chengdu.myqcloud.com/blog/15304180818347.jpg)

## 批量SN导入

![](https://media-1256569450.cos.ap-chengdu.myqcloud.com/blog/15302651139870.jpg)



## Lavarel-Admin的一些基础介绍

* Laravel是一个PHP的Web框架，中文文档：https://laravel-china.org/docs/laravel/5.5
* Laravel-Admin是基于Laravel的Admin后台管理工具，中文文档：http://laravel-admin.org/docs/#/zh/
* Valet是MacOS环境下可快速搭建Laravel环境的工具：https://github.com/laravel/valet
* Homestead是使用虚拟机来搭建Laravel环境的工具（Mac强烈建议Valet）：https://laravel-china.org/docs/laravel/5.5/homestead/1285

## 在Lavarel-Admin基础上使用Vue开发

我们看一下Laravel-Admin的源码目录：

```txt
├── app/
├── bootstrap/
├── config/
├── database/
├── node_modules/
├── public/
├── resources/
├── routes/
├── storage/
├── tests/
├── vendor/
├── artisan
├── composer.json
├── composer.lock
├── package.json
├── phpunit.xml
├── server.php
├── webpack.mix.js
└── yarn.lock
```

可以看到里边有一个文件`package.json`，玩过Node.js的都知道这个是用来描述有哪些js包的依赖和版本管理的一个文件。

打开`package.json`

![](https://media-1256569450.cos.ap-chengdu.myqcloud.com/blog/15304201808465.jpg)

可以看到，Laravel其实默认是允许使用Vue进行前端界面的开发了。

其中开发过程中，为了方便立即看到效果，一般会使用`yarn watch`进行监测js文件变化，并重生生成`public/js/app.js`

![](https://media-1256569450.cos.ap-chengdu.myqcloud.com/blog/15304204463607.jpg)

## 何处加载这个`app.js`

在文件`resources/views/admin/index.blade.php`

![](https://media-1256569450.cos.ap-chengdu.myqcloud.com/blog/15304205824947.jpg)

> 提示：Laravel-Admin作者提到可以使用`Admin::js()`进行加载js文件，但实际上我使用的并没有效果：https://github.com/z-song/laravel-admin/issues/267，因此，我这边就直接默认让它直接加载就好了。

基本的背景和知识点，介绍完毕，接下来讲述一下如何在Pjax将内容替换之后，再加载一些我们自己定制的Vue组件。

# 一种Pjax完成之后和再加载Vue组件的方法

拿「上传OTA升级包」为例，说明一下如何可以在`pjax`加载完成之后，再去加载一下我们自己的Vue组件。

**整体流程：**

- 打开网页时，已经加载好`app.js`和`pjax-end.js`
- 当用户切换了网页的路径之后
- pjax开始干活，当它完成网页内容替换之后，调用到`pjax-end.js`的js代码
- `pjax-end.js`就是负责根据路由，<font color=#f58220>来加载需要的Vue组件，以及清理不的Vue组件（不清可能会冲突</font>
- 当点击了我们使用blade模板写的按钮之后，<font color=#f58220>触发Vue组件的`data`修改（如从不可视切换到可视状态）</font>

在app.js里边有一段：

```js
function clean() {
  window.otapkg_upload = undefined
  window.ota_bulksn = undefined
}

// 加载上传OTA组件
function loadOtaUpload() {
  if (!window.otapkg_upload) {
    clean()
    if (document.getElementById("otapkg-upload")) {
      window.otapkg_upload = new Vue({
        el: "#otapkg-upload"
      })
      console.log(`loadOtaUpload done!`)
    }
  }
}

window.loadOtaUpload = loadOtaUpload
```

模板文件：`resources/views/onecloudpro/ota-pkg-upload.blade.php`

```php
<meta name="csrf-token" content="{{ csrf_token() }}">
<a class="btn btn-sm bg-olive" onclick="onUploadOtaClick();">上传升级包</a>

<div id='otapkg-upload'>
    <otapkg-upload-component></otapkg-upload-component>
</div>

<script>
    function onUploadOtaClick() {
        console.log("您点击了文件上传😘");
        var data = window.otapkg_upload.$children["0"].$data; // 一种获取Vue组件的data方式
        data.visiable = true;
    }
</script>
```

在`Collector`里边使用这个模板文件

```php
$grid->tools(function ($tools) {
    $tools->append(view('onecloudpro.ota-pkg-upload'));
});
```

编写`pjax-end.js`文件，让它在pjax完成之后，执行一些操作：

```js
$(document).on('ready pjax:end', function (event) {
    if (document.getElementById("otapkg-upload")) {
    	console.log('加载文件上传按钮组件')
        window.loadOtaUpload()
    }
    else if (document.getElementById("ota-bulksn")) {
    	console.log('加载批量SN导入按钮组件')
        window.loadOtaRuleBulkSn()
    }
});
```

整体搞好之后，效果如下：

![QQ20180702-101539-HD](https://media-1256569450.cos.ap-chengdu.myqcloud.com/blog/QQ20180702-101539-HD.gif)


> PS: 中途还尝试了使用`onhashchange`来监听路由的变化，但实际上与`pjax`是有冲突的。目前我们的CMDB系统承载了很多业务，并且前端后端揉合到一起了，如果可以从零开始，我可能会选择使用[vue-element-admin](https://github.com/PanJiaChen/vue-element-admin)来重构这个前端的界面~~



