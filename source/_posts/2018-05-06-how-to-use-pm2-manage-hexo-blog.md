---
layout: post
title: "如何使用pm2来管理hexo博客"
description: ""
category: 技术
tags: [pm2, hexo]
---


最近有想把博客部署到腾讯云上去，以加速在国内的访问速度。发现博客的启动依赖于`hexo server`这一个命令了。

我的基本思路是本地监听一个`127.0.0.1:4000`，然后再使用nginx的`proxy_pass`将请求转发给本地的服务，这样子还可以使用nginx来配置一下https请求。（题外话：虽然node.js也可以使用https，但那个性能真的与nginx的差得太多...）

<!-- more -->

这个是我的nginx配置`/etc/nginx/conf.d/blog.linkscue.com.conf`:

```conf
server {
    listen 80;
    server_name blog.linkscue.com;
    return 301 https://$host$request_uri;
    # location / {
    #     proxy_pass http://127.0.0.1:4000;
    # }
}

server {
    listen 443 ssl;
    server_name blog.linkscue.com;
    root /;
    charset utf-8;
    client_max_body_size 128M;

    ssl_certificate /etc/nginx/.ssl/full_chain.pem;
    ssl_certificate_key /etc/nginx/.ssl/private.key;

    ssl_session_cache shared:SSL:1m;
    ssl_session_timeout  10m;
    ssl_ciphers ALL:!kEDH!ADH:RC4+RSA:+HIGH:+EXP;
    ssl_prefer_server_ciphers on;

    location / {
    	proxy_pass http://127.0.0.1:4000;
    }
}
```

> PS: 我这样子的配置无论即使访问的是http也会强制转为https请求。

题外话就讲这么多，那怎么配置使用pm2来启动hexo呢，其实很简单，只要使用`pm2 start --name blog hexo -- s`就可以正常启动`hexo s`命令了。

那，`--name blog`又表示什么呢? 

表示，表示这个App Name是`blog`，一般情况下，服务器可能会同时使用pm2来启动多个服务，有了它才可以方便地区别各个服务。

![](https://ws1.sinaimg.cn/large/6e22ca27gy1fr20in2eyrj213w07kach)

这样子要记住这些命令还是有难度的，不妨我们将它写到`package.json`配置文件里边吧。

打开你的`package.json`配置文件，往里边添加配置:

```js
  "scripts": {
    "start": "pm2 start --name blog hexo -- s -i 127.0.0.1",
    "stop": "pm2 stop blog",
    "delete": "pm2 delete blog",
    "status": "pm2 status blog",
    "restart": "pm2 restart blog"
  },
```

这样子就可以使用`yarn start`来启动了，同样的，你还可以使用以下命令：

- yarn stop: 停止blog
- yarn restart: 重启blog
- yarn status: 查看blog的状态
- yarn delete: 清除运行的blog实例

同样的，我还回复了 https://github.com/hexojs/hexo/issues/2362 这个问答，来点小心心支持一下呀~

