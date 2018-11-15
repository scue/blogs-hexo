---
layout: post
title: "JS读取zip文件内的一个文件"
description: ""
category: 技术
tags: [js,zip]
---


开发前端Vue组件涉及OTA文件上传的时候，想优先通过读取OTA升级包(zip)文件的一个描述文件`description.json`来获取一些参数的信息，这样子不需要跑到后台去解析zip文件内容了。

<!-- more -->

```js
const jszip = require("jszip");
/**
 * OTA升级文件格式
 *
 * Archive: ota.zip
 * Length Date Time Name
 * --------- ---------- ----- ----
 * 75586 11-14-2018 21:32 firmware.hex
 * 124 11-14-2018 21:29 description.json
 * --------- -------
 * 75710 2 files
 */
/**
 * 解压获得OTA升级包内的描述信息
 * @param {Blob} file Zip文件打开之后的Stream
 * @param {string} desc_name Zip内包含描述信息的文件名
 * @return {Promise<string>}
 */
module.exports = function(file, desc_name) {
  return new Promise((resolve, reject) => {
    jszip
      .loadAsync(file)
      .then(zip => zip.file(desc_name).async("string"))
      .then(buffer => buffer.toString())
      .then(content => {
        console.log(`${desc_name} content is:\n${content}`);
        resolve(content);
      })
      .catch(e => {
        reject(`load zip file error: ${e}`);
      });
  });
};
```

依赖于`jszip`，使用yarn安装即可。

参考链接：
1. [Extracting zipped files using JSZIP in javascript](https://stackoverflow.com/questions/39322964/extracting-zipped-files-using-jszip-in-javascript)
2. [How to read a file](http://stuk.github.io/jszip/documentation/howto/read_zip.html)
