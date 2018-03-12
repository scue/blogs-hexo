---
layout: post
date: 2018-02-08 00:00:00
title: "Android计算字符串的MD5值"
description: ""
category: 技术
tags: 
  - Android
  - md5sum
---

之前找过一些版本，发现有一些使用场景下，md5sum会少一个字符，这对于一个已发布出去的版本是有一定的硬伤的。

```java
public static String md5sum(String str) {
        MessageDigest mdEncoder;
        try {
            mdEncoder = MessageDigest.getInstance("MD5");
            mdEncoder.update(str.getBytes(Charset.forName("US-ASCII")), 0, str.length());
            StringBuilder hexString = new StringBuilder();
            for (byte b : mdEncoder.digest()) {
                hexString.append(String.format("%02x", b&0xff));
            }
            return hexString.toString();
        } catch (NoSuchAlgorithmException e) {
            e.printStackTrace();
        }
        return null;
    }
```

