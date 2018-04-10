---
layout: post
title: "使用Node.js来写一个文件上传服务器"
description: "一个可以接收curl --upload-file文件上传的服务器"
category: 技术
tags: [Node.js, KOA]
---


# 背景描述

现在的项目越来越多，要求将玩客云小矿机进行日志文件上传的需要也越来越多。

通过这一篇文章，我们可以学习到如果使用Node.js来处理文件上传，包含兼容一下旧的`curl --upload-file`。

# 如何上传

一般我会使用如下命令：

```sh
curl -F 'files=@log.tgz' https://upload-logs.myserver.com/upload
```

通常以上的命令就足够了，但对于兼容性不太好的一些平台，可能使用这个命令不能正常上传，我会改用如下命令：

```sh
curl -T XXX.log -H "filename: XXX.log" https://upload-logs.myserver.com/upload
```

第一个命令，默认是走POST方式，而第二个命令走的是PUT方式，我们来分别写一个对应的代码：

## POST方式

```js
const mkdirp = require("mkdirp");
const fs = require("fs-extra");
const config = require("config");
const multer = require("koa-multer");
const moment = require("moment");
const Router = require("koa-router");
const logger = require("./logger");

// save path
const save_path = "/tmp/upload-logs";
mkdirp.sync(save_path);

// config upload storage
const storage = multer.diskStorage({
  destination: function(req, file, cb) {
    cb(null, save_path);
  },
  filename: function(req, file, cb) {
    const time = moment().format("YYYY-MM-DD_HH-mm-ss");
    cb(null, `${time}__${file.originalname}`);
  }
});
const upload = multer({ storage: storage });

router.post(
  "/upload",
  upload.fields([{ name: "files", maxCount: 1 }]),
  async (ctx, next) => {
    const log = ctx.req.files.files[0];
    logger.info(
      `save a file: ${log.originalname}, size: ${log.size} => ${log.path}`
    );
    ctx.body = {
      message: "OK"
    };
  }
);
```

其实这个很容易，主要是使用了`koa-multer`来处理，网上到处都有类似的代码，这里就不再阐述了。


## PUT方式

PUT方式比较复杂一点，也花费了我不少的时间去处理这个问题，先上代码：

```js
router.put("/upload", async (ctx, next) => {
  const time = moment().format("YYYY-MM-DD_HH-mm-ss");
  let filename =
    (ctx.headers.filename &&
      decodeURIComponent(escape(ctx.headers.filename))) ||
    "unknown.bin";
  let savefile = `${save_path}/${time}__${filename}`;
  logger.info(`Received a file: ${filename}, length: ${ctx.request.length}`);
  fs.removeSync(savefile);

  // recv data from socket
  ctx.req.socket.on("data", async data => {
    await fs.appendFileSync(savefile, data);
    // logger.info("appended +", data.length);
  });
  ctx.req.socket.on("end", () => {
    logger.info(`Saved to ${savefile}`);
  });
  ctx.body = {
    message: "OK"
  };
});
```

我花费了好长的时间去看koa的文件，发现`ctx.request`和`ctx.req`根本没有我想到的数据，只看到了一个长度`ctx.header.length`。

随后我想它是不是还没有把数据收上来，所以我就注意了一下`ctx.req.socket`这一个变量，然后尝试去`read`它，发现`read()`方法返回的也是null，完全没有数据。

随后我在官网上找到了一个关于socket的例子 https://nodejs.org/api/net.html#net_net_createconnection_options_connectlistener 发现人家是使用事件监听的机制去读取socket里边的内容的。

于是就有了如下的代码：

```js
ctx.req.socket.on("data", async data => {
    await fs.appendFileSync(savefile, data);
    // logger.info("appended +", data.length);
});
```

All Done，希望这个文章能帮助只在阅读的你。


