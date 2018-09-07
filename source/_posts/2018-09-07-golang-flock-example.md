---
layout: post
title: "Go语言巧用文件锁避免多个进程同时存在的问题"
description: ""
category: 技术
tags: [go, flock]
---

现实情况是你所开发的程序有可能被意外地多次拉起，而使用文件锁的排他锁功能可以解决这个问题。

文件锁有几个好处：

1. 避免多个进程同时存在
2. 程序意外中断，文件锁会自动解锁，而不自己亲自去收拾残局

使用起来也很简单，只需要指定一个lock文件，然后使用`syscall.Flock()`去上锁和解锁即可。

<!-- more -->

简单的，可以像这样子使用：

```go
func main() {
   // 打开文件锁
   lock, e := os.Create(LockFile)
   if e != nil {
      log.Printf("main: 创建文件锁失败: %s", e)
      os.Exit(ExitStatusLockCreateError)
   }
   defer os.Remove(LockFile)
   defer lock.Close()

   // 尝试独占文件锁
   e = syscall.Flock(int(lock.Fd()), syscall.LOCK_EX|syscall.LOCK_NB)
   if e != nil {
      log.Printf("main: 独占文件锁失败: %s", e)
      os.Exit(ExitStatusLockExError)
   }
   defer syscall.Flock(int(lock.Fd()), syscall.LOCK_UN)

   // your code ...
}
```

很多时候，像上边这样子就可以了， 如果觉得自己想封装一下，也是OK的：


```go
package flock

import (
	"errors"
	"os"
	"syscall"
)

type Flock struct {
	LockFile string
	lock     *os.File
}

// 创建文件锁，配合 defer f.Release() 来使用
func Create(file string) (f *Flock, e error) {
	if file == "" {
		e = errors.New("cannot create flock on empty path")
		return
	}
	lock, e := os.Create(file)
	if e != nil {
		return
	}
	return &Flock{
		LockFile: file,
		lock:     lock,
	}, nil
}

// 释放文件锁
func (f *Flock) Release() {
	if f != nil && f.lock != nil {
		f.lock.Close()
		os.Remove(f.LockFile)
	}
}

// 上锁，配合 defer f.Unlock() 来使用
func (f *Flock) Lock() (e error) {
	if f == nil {
		e = errors.New("cannot use lock on a nil flock")
		return
	}
	return syscall.Flock(int(f.lock.Fd()), syscall.LOCK_EX|syscall.LOCK_NB)
}

// 解锁
func (f *Flock) Unlock() {
	if f != nil {
		syscall.Flock(int(f.lock.Fd()), syscall.LOCK_UN)
	}
}
```

封装了之后，在`main()`函数上的调用可以像这样子：

```go
func main() {
	// 打开文件锁
	lock, e := flock.Create(LockFile)
	if e != nil {
		// handle error
	}
	defer lock.Release()

	// 尝试独占文件锁
	e = lock.Lock()
	if e != nil {
		// handle error
	}
	defer lock.Unlock()
	// your code ...
}
```

