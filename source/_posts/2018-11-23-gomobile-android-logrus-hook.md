---
layout: post
title: "gomobile: Hook logrus日志输出至android logcat"
description: ""
category: 技术
tags: [gomobile, cgo, android]
---

实际开发过程中，很多项目使用的日志输出是[logrus](https://github.com/sirupsen/logrus)，为了保持统一，同时又能把日志输出到Android Logcat，同时还带上Android的日志级别。

> 默认的log.Println，也能输出，但默认的Tag是`GoLog`，日志级别是`info`，显然不能满足要求。

<!-- more -->

定义文件: `log_android.go`
```go
package g

/*
#cgo LDFLAGS: -landroid -llog

#include <android/log.h>
#include <string.h>
#include <stdlib.h>
*/
import "C"
import (
	"unsafe"

	"github.com/Sirupsen/logrus"
)

var levels = []logrus.Level{
	logrus.PanicLevel,
	logrus.FatalLevel,
	logrus.ErrorLevel,
	logrus.WarnLevel,
	logrus.InfoLevel,
	logrus.DebugLevel,
}

type androidHook struct {
	tag *C.char
	fmt logrus.Formatter
}

type androidFormatter struct{}

func (f *androidFormatter) Format(entry *logrus.Entry) ([]byte, error) {
	return []byte(entry.Message), nil
}

func (hook *androidHook) Levels() []logrus.Level {
	return levels
}

func (hook *androidHook) Fire(e *logrus.Entry) error {
	var priority C.int

	formatted, err := hook.fmt.Format(e)
	if err != nil {
		return err
	}
	str := C.CString(string(formatted))

	switch e.Level {
	case logrus.PanicLevel:
		priority = C.ANDROID_LOG_FATAL
	case logrus.FatalLevel:
		priority = C.ANDROID_LOG_FATAL
	case logrus.ErrorLevel:
		priority = C.ANDROID_LOG_ERROR
	case logrus.WarnLevel:
		priority = C.ANDROID_LOG_WARN
	case logrus.InfoLevel:
		priority = C.ANDROID_LOG_INFO
	case logrus.DebugLevel:
		priority = C.ANDROID_LOG_DEBUG
	}
	C.__android_log_write(priority, hook.tag, str)
	C.free(unsafe.Pointer(str))
	return nil
}

// create a logrus Hook that forward entries to logcat
func AndroidLogHook(tag string) logrus.Hook {
	return &androidHook{
		tag: C.CString(tag),
		fmt: &androidFormatter{},
	}
}
```

使用方法：
```go
logrus.AddHook(g.AndroidLogHook())
```

然后就可以开心地使用`logrus.Debugf`等接口了，很方便，不是吗。

参考链接:
1. [logrus-android](https://github.com/bclermont/logrus-android)