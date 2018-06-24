---
layout: post
title: "关于使用Go开发Android和iOS底层代码，你想了解的都在这。"
description: ""
category: 技术
tags: [go, ios, android, gobind]
---


先来讲一下使用Go语言开发Android和iOS底层代码的好处：

- 可以编译成静态的`libgojni.so`或`<PKG>.framework`
- 跨平台代码复用率极高
- 二进制程序的安全性很好
- 代码可维护性很好

本文章介绍的内容：

1. 如何传递Go语言的'对象'至目标平台的语言
2. 如何传递目标平台语言至Go语言
3. 可穿越语言边界的数据类型，以及如何传递复杂类型
4. 如何在Go语言中使用目标平台语言已有的package
5. 如何有针对性的区分`android`和`ios`GO语言代码

<!-- more -->

本文章合适谁看？

合适已初步了解`gomobile`和`gobind`的童鞋，并想深入了解`gobind`的一些细节的人非常合适看本文章。

官网链接：https://godoc.org/golang.org/x/mobile/cmd/gobind

# 如何传递Go语言的'对象'至目标平台的语言

其实GO语言没有'对象'的说法，毕竟不存在类（class），所以，这里所述的'对象'其实是指Go语言的结构体的实例。

我们写一个`counter.go`文件：

```go
package hello

const (
	defaultValue = 10
)

type Counter struct {
	Value int
}

func (c *Counter) Inc() {
	c.Value++
}

func (c *Counter) Dec() {
	c.Value--
}

func NewCounter() *Counter {
	return &Counter{defaultValue}
}
```

目录结构：

```txt
├── android/
├── hello/
│   └── counter.go
└── ios/
```

Android代码：

```java
import hello.Counter;
import hello.Hello;
// ...
Counter counter = Hello.newCounter();
counter.inc();
Log.i(TAG, "onCreate: counter value " + counter.getValue());
```

Output: `06-24 21:13:19.431 28611-28611/com.linkscue.gobind.example I/MainActivity: onCreate: counter value 11`

iOS代码：

```objc
@import Hello;  // Gomobile bind generated framework
// ...
HelloCounter *counter = HelloNewCounter();
[counter inc];
NSLog(@"counter value %ld", counter.value);
```

Output: `2018-06-24 21:11:17.758095+0800 bind[16911:48934206] counter value 11`

> 解析：这里使用GO语言定义了`Counter`结构体，并提供了`NewCounter()`来生成'对象'，然后在目标平台上就可以直接调用它来生成一个'对象'了，接着在目标平台代码上可以使用这个'对象'提供的一些方法。

# 如何传递目标平台语言至Go语言

这个的前提是自己先知道在Go语言底下将使用目标代码的哪些接口，因此，需要事先在Go语言下定义好接口，比如:


```go
type Printer interface {
	Print(s string)
}
```

完整代码文件`printer.go`:

```go
package hello

import (
	"fmt"
	"time"
)

type Printer interface {
	Print(s string)
}

func PrintHello(p Printer) {
	p.Print(fmt.Sprintf(`Hello world! current time: %d`, time.Now().Unix()))
}
```

目录结构：

```txt
├── android/
├── hello/
│   ├── counter.go
│   └── printer.go
└── ios/
```

Android代码：

```java
Printer printer = new Printer() {
    @Override
    public void print(String s) {
        Log.w(TAG, "I print: " + s);
    }
};
Hello.printHello(printer); // 将printer传递至Golang
```

Output: `06-24 21:26:24.291 29190-29190/com.linkscue.gobind.example W/MainActivity: I print: Hello world! current time: 1529846784`

iOS代码：

- 头文件`SysPrinter.h`

```objc
#ifndef SysPrinter_h
#define SysPrinter_h

@import Hello;
@interface SysPrinter: NSObject<HelloPrinter>{}
@end

#endif /* SysPrinter_h */
```

- 源文件`SysPrinter.m`

```objc
#import "SysPrinter.h"
#import <Foundation/Foundation.h>

@implementation SysPrinter{}
- (void)print:(NSString *)s{
    NSLog(@"I print %@", s);
}
@end
```

- 调用方法

```objc
SysPrinter* printer = [[SysPrinter alloc] init];
HelloPrintHello(printer);
```

Output: `2018-06-24 21:30:46.722281+0800 bind[18140:48989176] I print Hello world! current time: 1529847046`

> 解析：在GO语言中定义了接口`Printer`接口，只要目标代码实现了`Print(s stirng)`即可。只不过这里的`Print(s string)`是Java或Objc实现的罢了，Go语言负责将所需要打印的字符串`s`生产出来，然后让java或objc代码来消费它。

# 可穿越语言边界的数据类型（type）

了解哪些类型（type）可以穿越语言边界很有用，可以让在设计的时候，就避开了很多坑。

- 有符号的整形和浮点数
- 字符串和布尔值（字符串映射成`String`或`NSString*`）
- 字节切片（`[]byte`），字节切片穿越边界之后是允许修改其中内容的
- 函数：任何使用上述类型的函数，没有返回值或只有一个返回值，若有两个返回值时，第二个参数必须是内置的error类型
- 接口：任何包含了符合上述条件的函数的接口
- 结构体：任何包含符合上述条件类型的结构体都能穿越语言边界

官网：https://godoc.org/golang.org/x/mobile/cmd/gobind#hdr-Type_restrictions

## 传递复杂类型

可以看到，允许穿越语言边界的类型是比较少的，像List和Map比较常用的，建议可以通过Json序列化来传递。

比如，可以写一段这样子的Android调用gojni的代码：

Android 代码：

```java
// 通过Json传递复杂数据至JNI层
JSONArray cmds = new JSONArray();
cmds.add("echo");
cmds.add("hello");
String command = cmds.toJSONString(); // ["echo", "hello"]
Request request = Hello.newRequest(command, false, 0);
```

GO语言代码：

```go
type Request struct {
	Uid        int      `json:"uid"`        // 调用者的UID
	Command    []string `json:"command"`    // 执行的命令
	Background bool     `json:"background"` // 后台运行
	Timeout    int      `json:"timeout"`    // 超时时间，秒
	Timestamp  int64    `json:"timestamp"`  // 时间戳，秒
}

func NewRequest(command string, background bool, timeout int) *Request {
   // 解析传递下来的Json串
	var cmdArray []string
	e := json.Unmarshal([]byte(command), &cmdArray)
	if e != nil {
		log.Println(`NewRequest: json decode error:`, e)
		return nil
	}
	return &Request{
		Uid:        os.Getuid(),
		Command:    cmdArray,
		Background: background,
		Timeout:    timeout,
		Timestamp:  time.Now().Unix(),
	}
}
```

> 解析：像`[]string`字符串数组（切片）是不允许穿越语言边界的，但使用Json序列化成String之后就不存在限制了。

# 如何在Go语言中使用目标平台语言已有的package

在GO语言中，还可以像`Cgo`一样，直接使用已存在的package，这里的奇技淫巧不一定能使用得上，不过我们可以先了解一下。

假设：现在希望使用各自平台的目标代码来获取当前系统的Unix时间。

已知：Android，可以使用`System.CurrentTimeMillis()`来获取，iOS，可以使用`NSDate.date.timeIntervalSince1970`来获取。

目标结构：

```txt
├── android/
├── hello/
│   ├── counter.go
│   ├── hello.go
│   ├── printer.go
│   ├── request.go
│   ├── reverse_android.go // Android 反向使用Java代码
│   └── reverse_ios.go     // iOS 反向使用Objc代码
└── ios/
```

> 提示：需要使用`_android`和`_ios`来区分各平台的代码，一方面是清晰，另一方面是不这样子做编译不过...

Android的`reverse_android.go`:

```go
package hello

import "Java/java/lang/System"

func reverseCurrentTime() uint64 {
	return uint64(System.CurrentTimeMillis() / 1000)
}
```

iOS的`reverse_ios.go`:

```go
// +build ios

package hello

import (
	"ObjC/Foundation/NSDate"
)

func reverseCurrentTime() uint64 {
	return uint64(NSDate.Date().TimeIntervalSince1970())
}
```

> 提示：`// +build ios`这行注释是必须的，说明仅在编译的tags包含了`ios`的时候才会编译这个文件，而Android则不需要`// +build android`这个注释也可以。同时，这一行的注释必须是在`package hello`之前，必须包含一行空白行。

# 如何有针对性的区分`android`和`ios`GO语言代码

上一小节已说明得比较清楚了，这里复述一次：

- 使用`_android`和`_ios`文件命名方式来区分不同平台的代码
- 在`_ios`文件上，必须包含`// +build ios`注释
- 这一行注释必须在`package hello`之前，并预留一行空白行

> 提示：在`ios/`目录下执行 `gomobile bind -a -v -target=ios ../hello`命令时，实际调用go build的命令是这样子的`go build -tags ios -v -buildmode=c-archive -o /path/to/hello-arm.a`，编译`ios`的时候使用附加上`-tags ios`。

有兴趣阅读：https://golang.org/pkg/go/build/#hdr-Build_Constraints

# 阅后福利

## 不应在go语言`init()`反向使用目标代码的package

不然会出现以下错误：

- Android错误日志：

```log
06-24 22:18:18.611 31929-31943/com.linkscue.gobind.example E/Go: panic: runtime error: invalid memory address or nil pointer dereference
    [signal SIGSEGV: segmentation violation code=0x1 addr=0x0 pc=0x608c6f28]
    goroutine 1 [running]:
    github.com/scue/gobind-example/hello.init.0()
    	/Users/scue/go/src/github.com/scue/gobind-example/hello/reverse_android.go:13 +0x1c
06-24 22:18:18.611 31929-31943/com.linkscue.gobind.example A/libc: Fatal signal 6 (SIGABRT) at 0x00007cb9 (code=-6), thread 31943 (.gobind.example)
```


- iOS错误日志:

```log
Thread 2: EXC_BAD_ACCESS (code=1, address=0x0)
```

## 使用gobind的示例iOS `textLabel.text`设定之后无效？

![](https://media-1256569450.cos.ap-chengdu.myqcloud.com/blog/15298504040784.jpg)

定睛一看，发现这个Label很皮，跑到左上角去了，把它拖下来即可。

