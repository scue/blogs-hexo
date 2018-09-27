---
layout: post
title: "使用C语言来写一个Python扩展库"
description: ""
category: 技术
tags: [c, python]
---

一些加密解密功能比较重要，底层的实现不应该被外界的人所感知。

跟随我的步伐，花费10分钟的阅读时间，三步之内解决使用C语言来写扩展库的问题（python 2.7）。

<!-- more -->

# 目录文件
```txt
├── crypt.h      // 头文件
├── libcrypt.a   // 包含Encrypt和Decrypt的具体实现
├── occrypt.c    // 暴露给Python使用的主要文件
├── occrypt.so*  // 生成的Python扩展库
└── test.py      // 测试程序
```

# C语言写Python扩展库示例

**`occrypt.c`文件内容**
```c
#include "crypt.h"
#include <Python.h>
#include <string.h>
#include <stdio.h>
#include <stdlib.h>
static PyObject * /* returns string object */
oc_encrypt(PyObject *self, PyObject *args)
{
    char *key = NULL;
    char *plainText = NULL;
    if (!PyArg_ParseTuple(args, "ss", &key, &plainText)) /* convert Python -> C */
        return NULL; /* null=raise exception */
    char *text = Encrypt(key, plainText);
    PyObject *ret = Py_BuildValue("s", text); /* convert C -> Python */
    free(text);
    return ret;
}
static PyObject * /* returns string object */
oc_decrypt(PyObject *self, PyObject *args)
{
    char *key = NULL;
    char *plainText = NULL;
    if (!PyArg_ParseTuple(args, "ss", &key, &plainText)) /* convert Python -> C */
        return NULL; /* null=raise exception */
    char *text = Decrypt(key, plainText);
    PyObject *ret = Py_BuildValue("s", text); /* convert C -> Python */
    free(text);
    return ret;
}
/* registration table */
static PyMethodDef crypt_methods[] = {
    {"encrypt", oc_encrypt, METH_VARARGS, "encrypt text"}, /* method name, C func ptr, always-tuple */
    {"decrypt", oc_decrypt, METH_VARARGS, "decrypt text"}, /* method name, C func ptr, always-tuple */
    {NULL, NULL, 0, NULL} /* end of table marker */
};
/* module initializer */
PyMODINIT_FUNC initoccrypt() /* called on first import */
{ /* name matters if loaded dynamically */
    (void)Py_InitModule("occrypt", crypt_methods); /* mod name, table ptr */
}
```

**相关解释与说明**

- `initoccrypt`函数的命名规则是`init<name>`
- 相应的动态库的名字`occrypt.so`命名规则也是<name>.so
- `oc_encrypt`和`oc_decrypt`是对`Encrypt`和`Decrypt`的进一步封装，使它可以被Python代码调用
- 使用`PyArg_ParseTuple`来解析传入的Python参数
- 使用`Py_BuildValue("s", text)`来返回一个字符串
- 关于`PyArg_ParseTuple`和`Py_BuildValue`参数规则可以访问：https://docs.python.org/2/c-api/arg.html#c.Py_BuildValue

# 编译C语言扩展库示例

编译方法：
```sh
clang occrypt.c libcrypt.a \
    -I/usr/local/anaconda3/envs/py27/include/python2.7 \
    -L/usr/local/anaconda3/envs/py27/lib/  -lpython2.7 \
    -shared -fPIC -o occrypt.so
```

**相关解释与说明**

- `-I/usr/local/anaconda3/envs/py27/include/python2.7`表示Include路径，这个路径包含了一个`Python.h`
- `-L/usr/local/anaconda3/envs/py27/lib/`表示ld链接路径，这个路径包含了一个`libpython2.7.dylib`

请根据自己的环境做相应的调节

> 提示：`clang`是macos的编译工具，linux请切换至`gcc`

# Python使用C扩展库示例

**`test.py`文件内容**

```py
#!/usr/bin/env python
# -*- coding: UTF-8 -*-
import occrypt
text = occrypt.encrypt("secret-key", "hello go go go!")
print("加密文本：%s" % text)
print("解密文本: %s" % occrypt.decrypt("secret-key", text))
```
进入含有`occrypt.so`的目录，调用`python test.py`的输出结果：

```txt
加密文本：5iDfvYIH9l7oTXaGrpbqCH1z24zJHjBaK9wTMQHlxA==
解密文本: hello go go go!
```

参考链接
1. [15.1 使用ctypes访问C代码](https://python3-cookbook.readthedocs.io/zh_CN/latest/c15/p01_access_ccode_using_ctypes.html)
2. [Python调用C/C++的种种方法](https://blog.csdn.net/fxjtoday/article/details/6059874)
3. [python使用 C语言类型、ctypes 的用法](https://blog.csdn.net/eleanoryss/article/details/70331973)
4. [Extending Python with C or C++](https://docs.python.org/2/extending/extending.html)
5. [Parsing arguments and building values](https://docs.python.org/2/c-api/arg.html#c.Py_BuildValue)
6. [python和C语言混合编程实例](https://m.jb51.net/show/50633)
7. [gcc编译参数-fPIC的一些问题](http://blog.sina.com.cn/s/blog_54f82cc201011op1.html)