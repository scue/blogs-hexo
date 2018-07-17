---
layout: post
title: "Nginx配置——对一些路由进行IP访问限制"
description: ""
category: 技术
tags: [Nginx]
---

现实中，你有总有一些路由是针对内部人员才开放的，比如重新加载配置、更新配置等操作。

本文将告诉你如何限制某一些路由仅限于内部的IP可以访问，而其他访问则是403 Forbidden。

<!-- more -->

Nginx配置文件：

```conf
#
# HTTPS server configuration
#

server {
    listen 80;
    server_name ops-conf.onething.net;
    return 301 https://$host$request_uri;
    # location / {
    #     proxy_pass http://127.0.0.1:4430;
    # }
}

server {
    listen 443 ssl;
    server_name someweb.example.net;
    root /;
    charset utf-8;
    client_max_body_size 128M;

    ssl_certificate /usr/local/openresty/nginx/ssl/some.crt;
    ssl_certificate_key /usr/local/openresty/nginx/ssl/some.key;

    ssl_session_cache shared:SSL:1m;
    ssl_session_timeout  10m;
    ssl_ciphers ALL:!kEDH!ADH:RC4+RSA:+HIGH:+EXP;
    ssl_prefer_server_ciphers on;

    location ~ ^/reload|/some-feature/reload.* {
        proxy_pass http://127.0.0.1:1234;
        allow 36.xxx.yyy.112;
        deny  all;
    }

    location / {
        proxy_pass http://127.0.0.1:1234;
    }
}
```

其中主要的是安全限制配置如下：

```conf
location ~ ^/reload|/some-feature/reload.* {
        proxy_pass http://127.0.0.1:1234;
        allow 36.xxx.yyy.112;
        deny  all;
    }
```

即只允许`36.xxx.yyy.112`这个IP可以访问得到，其他都拒绝访问。

* 特定机器的访问

  ![](https://media-1256569450.cos.ap-chengdu.myqcloud.com/blog/15317930693147.jpg)

* 其他机器的访问

  ![](https://media-1256569450.cos.ap-chengdu.myqcloud.com/blog/15317931343801.jpg)

通过这种简单的Nginx配置，就可以进行IP访问限制了。