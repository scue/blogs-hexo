---
layout: post
title: "GO语言AES加密"
description: ""
category: 技术
tags: [GO, AES]
---

平时工作难免会有一些信息是非常重要的，不期望被他人给明文窃取，尤其是一些重要的资料进行落地的时候，需要进行一下加密。

<!-- more -->

```go
// 加密
func encrypt(key []byte, text string) (cryptText string, err error) {
    plainText := []byte(text)
    block, err := aes.NewCipher(key)
    if err != nil {
        return
    }
    cipherText := make([]byte, aes.BlockSize+len(plainText))
    iv := cipherText[:aes.BlockSize]
    if _, err = io.ReadFull(rand.Reader, iv); err != nil {
        return
    }
    stream := cipher.NewCFBEncrypter(block, iv)
    stream.XORKeyStream(cipherText[aes.BlockSize:], plainText)
    cryptText = base64.URLEncoding.EncodeToString(cipherText)
    return
}
// 解密
func decrypt(key []byte, cryptMsg string) (plainText string, err error) {
    cipherText, err := base64.URLEncoding.DecodeString(cryptMsg)
    if err != nil {
        return
    }
    block, err := aes.NewCipher(key)
    if err != nil {
        return
    }
    if len(cipherText) < aes.BlockSize {
        err = errors.New("加密内容太短无法进行解密")
        return
    }
    iv := cipherText[:aes.BlockSize]
    cipherText = cipherText[aes.BlockSize:]
    stream := cipher.NewCFBDecrypter(block, iv)
    stream.XORKeyStream(cipherText, cipherText)
    plainText = string(cipherText)
    return
}
```

其中key是一个16字节的`[]byte`，不能太短，不然无法完成加密~

可以看到，这里使用了AES/CFB进行加密，如果要保证数据的安全，最为重要的是要保证Key不被他人给窃取。若加密/解密只在同一个设备上运行，这个Key最好与设备的某一些属性关联，一台被破解不会涉及其他机器。


