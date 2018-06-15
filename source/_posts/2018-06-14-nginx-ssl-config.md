---
layout: post
title: "Nginx配置https和端口映射"
description: ""
category: 技术
tags: [nginx, https]
---

安全的问题不容小觑，一般我所开发的后台服务器，我都会要求使用https，以减少网络的中间攻击导致不必要的损失。

一般情况下，我开发的后台，只是监听的是`127.0.0.1:4430`类似的端口，仅限本地访问。

然后通过nginx来处理外部的请求，将它转发至本地的服务上，顺带使用nginx来配置一下https的证书。

<!-- more -->

```txt
$ cat /etc/nginx/conf.d/www.exmaple.com.conf
#
# HTTPS server configuration
#

server {
    listen 80;
    server_name www.exmaple.com;
    return 301 https://$host$request_uri;
    # location / {
    #     proxy_pass http://127.0.0.1:4430;
    # }
}

server {
    listen 443 ssl http2;
    server_name  www.exmaple.com;
    root /;
    charset utf-8;
    client_max_body_size 128M;

    ssl_certificate /etc/nginx/ssl/example.com.crt;
    ssl_certificate_key /etc/nginx/ssl/example.com.key;

    ssl_session_cache shared:SSL:1m;
    ssl_session_timeout  10m;
    ssl_ciphers ALL:!kEDH!ADH:RC4+RSA:+HIGH:+EXP;
    ssl_prefer_server_ciphers on;

    location / {
    proxy_pass http://127.0.0.1:4430;
    }
}
```

- 服务器地址：`www.exmaple.com`
- 将Http自定向为Https
- 配置了ssl证书
- 将`www.exmaple.com`重定向到了本地的`4430`的应用

