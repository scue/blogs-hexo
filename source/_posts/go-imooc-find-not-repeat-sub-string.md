---
layout: post
date: 2018-03-03 00:00:00
title: "寻找最长不含有重复字符的子串"
description: ""
category: 技术
tags: 
  - go
---

寻找最长不含有重复字符的子串

<!-- more -->

```go
package main

import "fmt"

// 寻找最长不含有重复字符的子串（长度）
func maxLenOfNotRepeatString(s string) (maxLength int) {
	start := 0
	occurred := make(map[rune]int)

	// 须将字符串转为[]rune以支持utf8
	for i, ch := range []rune(s) {

		// 一旦发现有重复，将start位置重置为重复字符的下一位
		if lastI, ok := occurred[ch]; ok && lastI >= start {
			start = lastI + 1
		}

		// 检查并调整最大长度
		curLength := i - start + 1
		if curLength > maxLength {
			maxLength = curLength
		}

		// 记录此字符最后位置
		occurred[ch] = i
	}
	return maxLength
}

func main() {
	fmt.Println(maxLenOfNotRepeatString("abcab"))
	fmt.Println(maxLenOfNotRepeatString("bbbbb"))
	fmt.Println(maxLenOfNotRepeatString("天涯海角"))
	fmt.Println(maxLenOfNotRepeatString("芳草萋萋"))
	fmt.Println(maxLenOfNotRepeatString("芳草萋萋鹦鹉洲"))
}
```

输出结果：

```txt
3
1
4
3
4

Process finished with exit code 0
```


