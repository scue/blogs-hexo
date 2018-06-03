---
layout: post
title: "GO语言获取程序自身MD5值"
description: ""
category: 技术
tags: [GO]
---

获取自身的MD5值可能有很多用途，比如我们将程序运行客户端时，我们就可以通过对比MD5值来确保自己的程序没有被恶意地篡改。

<!-- more -->

# 二进制程序

一般情况下，golang编译出来的程序是二进制程序，就可以像这样子来获取：

```go
// 获取程序自身的MD5
func selfMd5sum() string {
   path, e := exec.LookPath(os.Args[0])
   if e != nil {
      log.Println(`Self md5sum error: look path fail:`, e)
      return ""
   }
   bs, e := ioutil.ReadFile(path)
   if e != nil {
      log.Println(`SelfMd5sum error: read file fail:`, e)
      return ""
   }
   sum := md5.Sum(bs)
   return hex.EncodeToString(sum[:])
}
```

> `exec.LookPath`是为了获取得到实际运行的路径

# 动态库

什么情况下会有动态库？比如我拿GO来开发Android的一些功能的时候，像gomobile bind的时候就会输出`libgojni.so`文件。

```go
// 获取程序自身的MD5
func SelfMd5sum() string {
	jniLibPath := fmt.Sprintf(`%s/libgojni.so`, JniDir)
	bs, e := ioutil.ReadFile(jniLibPath)
	if e != nil {
		log.Println(`SelfMd5sum error: read file fail:`, e)
		return ""
	}
	sum := md5.Sum(bs)
	return hex.EncodeToString(sum[:])
}
```

这个`jniLibPath`是通过java层上的`getApplicationInfo().nativeLibraryDir`传递下来的，一般像这样子：` /data/app/<package>-1/lib/arm64/`。





