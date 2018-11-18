---
layout: post
title: "JS读取cpio文件内的一个文件"
description: ""
category: 技术
tags: [js,cpio]
---


开发前端Vue组件涉及OTA文件上传的时候，想优先通过读取OTA升级包(cpio)文件的一个描述文件`description.json`来获取一些参数的信息，这样子不需要跑到后台去解析cpio文件内容了。

<!-- more -->


```js
const cpio = require("cpio-stream");
/**
 * 解压获得CPIO升级包内的描述信息
 * @param {stream.Readable} file CPIO文件打开之后的Stream
 * @param {string} desc_name CPIO内包含描述信息的文件名
 * @return {Promise<string>}
 */
module.exports = function(file, desc_name) {
  let extract = cpio.extract();
  return new Promise((resolve, reject) => {
    extract.on("entry", function(header, stream, next) {
      // header is the tar header
      // stream is the content body (might be an empty stream)
      // call next when you are done with this entry
      let body = "";
      // 读取流文件内容
      stream.on("data", function(chunk) {
        if (header.name == desc_name) {
          body += chunk;
        }
      });
      // 读取流文件结束
      stream.on("end", function() {
        if (header.name == desc_name) {
          resolve(body);
        }
        next();
      });
      stream.resume(); // just auto drain the stream
    });
    extract.on("finish", function() {
      reject("all entries already read");
    });
    file.pipe(extract);
  });
};
```

依赖于`cpio-stream`，使用yarn安装即可，前端调用的时候需要提供一个stream，正常前端读取文件的是Blob，可能你还需要使用`blob-to-stream`进行转换。

参考链接：
1. [blob-to-stream](https://www.npmjs.com/package/blob-to-stream)
2. [cpio-stream](https://www.npmjs.com/package/cpio-stream)