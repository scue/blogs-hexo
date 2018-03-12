---
layout: post
date: 2018-03-01 00:00:00
title: "GO指南 练习：等价二叉查找树"
description: ""
category: 技术
tags: 
  - go
---

1. 实现 `Walk` 函数。

2. 测试 `Walk` 函数。
    
    函数 `tree.New(k)` 用于构造一个随机结构的已排序二叉查找树，它保存了值 `k 、 2k 、 3k ... 10k` 。
    
    创建一个新的信道 `ch` 并且对其进行步进：
    
    `go Walk(tree.New(1), ch)`
    然后从信道中读取并打印 10 个值。应当是数字 `1, 2, 3, ..., 10` 。

3. 用 `Walk` 实现 `Same` 函数来检测 `t1` 和 `t2` 是否存储了相同的值。

4. 测试 `Same` 函数。

<!-- more -->

`Same(tree.New(1), tree.New(1))` 应当返回 `true` ，而 `Same(tree.New(1), tree.New(2))` 应当返回 `false` 。

Tree 的文档可在这里找到。

```go
package main

import (
	"fmt"
	"sort"

	"golang.org/x/tour/tree"
)

func walk(t *tree.Tree, ch chan int) {
	if t == nil {
		return
	}
	walk(t.Left, ch)
	ch <- t.Value
	walk(t.Right, ch)
}

// Walk 遍历 tree t 将所有的值从 tree 发送到 channel ch。
func Walk(t *tree.Tree, ch chan int) {
	walk(t, ch)
	close(ch)
}

// Same 检测树 t1 和 t2 是否含有相同的值。
func Same(t1, t2 *tree.Tree) bool {
	ch1 := make(chan int)
	ch2 := make(chan int)
	a1 := make([]int, 0)
	a2 := make([]int, 0)
	go Walk(t1, ch1)
	go Walk(t2, ch2)
	for v := range ch1 {
		a1 = append(a1, v)
	}
	for v := range ch2 {
		a2 = append(a2, v)
	}
	if len(a1) != len(a2) {
		return false
	}
	
	// 是否应当Sort？
	// 题目要求的是：检测 t1 和 t2 是否存储了相同的值
	sort.Ints(a1)
	sort.Ints(a2)
	for i, v := range a1 {
		if v != a2[i] {
			return false
		}
	}
	return true
}

func main() {

	// Test Walk
	ch := make(chan int)
	t := tree.New(1)
	fmt.Println("tree:", t)
	go Walk(t, ch)
	for v := range ch {
		fmt.Println("Got value:", v)
	}

	// Test Same
	t2 := tree.New(1)
	fmt.Println("Check same:", Same(t, t2))
}

```


程序输出：

```txt
tree: ((((1 (2)) 3 (4)) 5 ((6) 7 ((8) 9))) 10)
Got value: 1
Got value: 2
Got value: 3
Got value: 4
Got value: 5
Got value: 6
Got value: 7
Got value: 8
Got value: 9
Got value: 10
Check same: true
```


链接：https://tour.go-zh.org/concurrency/8

