---
layout: post
date: 2018-02-28 00:00:00
title: "GO指南 练习：Map"
description: ""
category: 技术
tags: 
  - go
---



实现 `WordCount` 。它应当返回一个映射，其中包含每个字符串 `s` 中“单词”的个数。函数 `wc.Test` 会对此函数执行一系列测试用例，并输出成功还是失败。

你会发现 `strings.Fields` 很有帮助。


```go
package main

import (
	"strings"

	"golang.org/x/tour/wc"
)

func WordCount(s string) map[string]int {
	a := strings.Fields(s)
	m := make(map[string]int)
	for _, v := range a {
		m[v]++
	}
	return m
}

func main() {
	wc.Test(WordCount)
}
```

输出结果：


```text
PASS
 f("I am learning Go!") = 
  map[string]int{"I":1, "am":1, "learning":1, "Go!":1}
PASS
 f("The quick brown fox jumped over the lazy dog.") = 
  map[string]int{"over":1, "the":1, "lazy":1, "dog.":1, "The":1, "brown":1, "fox":1, "jumped":1, "quick":1}
PASS
 f("I ate a donut. Then I ate another donut.") = 
  map[string]int{"I":2, "ate":2, "a":1, "donut.":2, "Then":1, "another":1}
PASS
 f("A man a plan a canal panama.") = 
  map[string]int{"plan":1, "canal":1, "panama.":1, "A":1, "man":1, "a":2}

Program exited.
```

链接：https://tour.go-zh.org/moretypes/23


