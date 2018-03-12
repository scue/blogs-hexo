---
layout: post
date: 2018-03-01 00:00:00
title: "GO指南 练习：Web爬虫"
description: ""
category: 技术
tags: 
  - go
---

在这个练习中，我们将会使用 Go 的并发特性来并行化一个 Web 爬虫。

修改 `Crawl` 函数来并行地抓取 URL，并且保证不重复。

<!-- more -->

_提示：_ 你可以用一个 `map` 来缓存已经获取的 `URL`，但是要注意 `map` 本身并不是并发安全的！

```go
package main

import (
	"fmt"
	"sync"
)

type Fetcher interface {
	// Fetch 返回 URL 的 body 内容，并且将在这个页面上找到的 URL 放到一个 slice 中。
	Fetch(url string) (body string, urls []string, err error)
}

// Crawl 使用 fetcher 从某个 URL 开始递归的爬取页面，直到达到最大深度。
func Crawl(url string, depth int, fetcher Fetcher) {
	// TODO: 并行的抓取 URL。
	// TODO: 不重复抓取页面。
	// 下面并没有实现上面两种情况：
	if depth <= 0 {
		return
	}
	
	// 避免重复抓取页面
	if checkUrl(url) {
		return
	}
	
	// 标记已抓取页面
	maskUrlCount(url, 1)
	body, urls, err := fetcher.Fetch(url)
	if err != nil {
		fmt.Println(err)
		// 题目：修改 Crawl 函数来并行地抓取 URL，并且保证不重复。
		// FIXME: 抓取失败是否需要标记未抓取？
		// maskUrlCount(url, -1)
		return
	}
	fmt.Printf("found: %s %q\n", url, body)

	// 并行的抓取URL
	ch := make(chan int)
	for _, u := range urls {
		go func(url string) {
			Crawl(url, depth-1, fetcher)
			ch <- 1
		}(u)
	}

	// 等待抓取完成
	for range urls {
		<-ch
	}

	return
}

var urlCounter UrlCounter

func checkUrl(url string) bool {
	urlCounter.mux.Lock()
	defer urlCounter.mux.Unlock()
	_, ok := urlCounter.v[url]
	// if ok && c == 0 {
	// 	return false
	// }
	return ok
}

func maskUrlCount(url string, count int) {
	urlCounter.mux.Lock()
	defer urlCounter.mux.Unlock()
	urlCounter.v[url] += count
}

func main() {
	urlCounter = UrlCounter{v: make(map[string]int)}
	Crawl("http://golang.org/", 4, fetcher)
}

// UrlCounter 的并发使用是安全的。
type UrlCounter struct {
	v   map[string]int
	mux sync.Mutex
}

// fakeFetcher 是返回若干结果的 Fetcher。
type fakeFetcher map[string]*fakeResult

type fakeResult struct {
	body string
	urls []string
}

func (f fakeFetcher) Fetch(url string) (string, []string, error) {
	if res, ok := f[url]; ok {
		return res.body, res.urls, nil
	}
	return "", nil, fmt.Errorf("not found: %s", url)
}

// fetcher 是填充后的 fakeFetcher。
var fetcher = fakeFetcher{
	"http://golang.org/": &fakeResult{
		"The Go Programming Language",
		[]string{
			"http://golang.org/pkg/",
			"http://golang.org/cmd/",
		},
	},
	"http://golang.org/pkg/": &fakeResult{
		"Packages",
		[]string{
			"http://golang.org/",
			"http://golang.org/cmd/",
			"http://golang.org/pkg/fmt/",
			"http://golang.org/pkg/os/",
		},
	},
	"http://golang.org/pkg/fmt/": &fakeResult{
		"Package fmt",
		[]string{
			"http://golang.org/",
			"http://golang.org/pkg/",
		},
	},
	"http://golang.org/pkg/os/": &fakeResult{
		"Package os",
		[]string{
			"http://golang.org/",
			"http://golang.org/pkg/",
		},
	},
}

```


程序输出：

```txt
found: http://golang.org/ "The Go Programming Language"
found: http://golang.org/pkg/ "Packages"
not found: http://golang.org/cmd/
found: http://golang.org/pkg/fmt/ "Package fmt"
found: http://golang.org/pkg/os/ "Package os"

Program exited.
```

**注意其中的并发写法**：

```go
go func(url string) {
    Crawl(url, depth-1, fetcher)
    ch <- 1
}(u)
```

==需要将`url`传入，而不能采用如下的写法==：


```go
go func() {
    Crawl(u, depth-1, fetcher)
    ch <- 1
}()
```

原因是起来之后都共用了同一个外层的`u`变量，而这个`u`变量是受到`for`循环的影响的，导致输出结果的减少。

链接：https://tour.go-zh.org/concurrency/10


