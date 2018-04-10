---
layout: post
title: "GO完全静态编译"
description: CGO_ENABLED=0 go build --ldflags "-extldflags -static"
category: 技术
tags: [go]
---



```sh
CGO_ENABLED=0 go build --ldflags "-extldflags -static" -o bin/frps-static ./cmd/frps
```

实际可以看到效果如下：

```txt
/go/src/github.com/fatedier/frp # go build --ldflags "-extldflags -static" -o bin/frps-static ./cmd/frps
/go/src/github.com/fatedier/frp # ldd bin/frps-static
    /lib/ld-musl-x86_64.so.1 (0x7fb254e7f000)
    libc.musl-x86_64.so.1 => /lib/ld-musl-x86_64.so.1 (0x7fb254e7f000)
/go/src/github.com/fatedier/frp # CGO_ENABLED=0 go build --ldflags "-extldflags -static" -o bin/frps-static ./cmd/frps
/go/src/github.com/fatedier/frp # ldd bin/frps-static
ldd: bin/frps-static: Not a valid dynamic program
```

`bin/frps-static` 就是静态的文件，再不也不是动态的了。


参考链接：

1. https://www.yryz.net/post/golang-docker-alpine-start-panic.html
2. http://blog.wrouesnel.com/articles/Totally%20static%20Go%20builds/
3. http://blog.cloud66.com/how-to-create-the-smallest-possible-docker-image-for-your-golang-application/


