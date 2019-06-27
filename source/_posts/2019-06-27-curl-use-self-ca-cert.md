---
layout: post
title: "Curl命令使用自建的CA证书"
description: ""
category: 技术
tags: [curl, ca]
---

我将在本文章介绍一种不需要在系统上安装证书，也可以验证我们的自建证书是否起作用的方法。


> Tips: 毕竟有时候给系统安装证书，总有一种被玷污的感觉--!

<!-- more -->


macOS原生自带的使用`--cacert`参数指定证书是不生效的，需要通过brew安装新的版本。

```sh
# ① 安装 curl
brew install curl
brew link curl --force

# ② 设定PATH环境变量，优先使用brew安装的curl
export PATH="/usr/local/opt/curl/bin:$PATH"

# ③ 重新写一个受信任CA证书列表文件
cat /usr/local/etc/openssl/cert.pem ca.crt > cert_bundle.pem

# ④ 指定使用这些证书去访问网站
curl -vL --cacert ./cert_bundle.pem https://example.org

# ⑤ 更神奇的是，还可以指定域名解析
curl -vL --cacert ./cert_bundle.pem --resolve example.org:443:127.0.0.1 https://example.org
```

详细输出如下：

```
$ curl -vL --cacert ./cert_bundle.pem --resolve example.org:443:127.0.0.1 https://example.org
* Added example.org:443:127.0.0.1 to DNS cache
* Hostname example.org was found in DNS cache
*   Trying 127.0.0.1:443...
* TCP_NODELAY set
* Connected to example.org (127.0.0.1) port 443 (#0)
* ALPN, offering http/1.1
* TLS 1.2 connection using TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
* Server certificate: *.exmaple.org
* Server certificate: Linkscue Secondary CA
* Server certificate: Linkscue Root CA
> GET / HTTP/1.1
> Host: example.org
> User-Agent: curl/7.65.0
> Accept: */*
>
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< Server: nginx/1.17.0
< Date: Thu, 27 Jun 2019 06:52:02 GMT
< Content-Type: text/html; charset=utf-8
< Content-Length: 30
< Last-Modified: Fri, 21 Jun 2019 11:46:48 GMT
< Connection: keep-alive
< ETag: "5d0cc3a8-1e"
< Accept-Ranges: bytes
<
<h1>Welcome example.org!</h1>
* Connection #0 to host example.org left intact
```