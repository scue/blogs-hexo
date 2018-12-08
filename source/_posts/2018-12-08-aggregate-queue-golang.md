---
layout: post
title: "GO语言一种按秒聚合消息队列的实现方法"
description: ""
category: 技术
tags: [go]
---

Linux内核大概只有100万次每秒的收发网络数据包的能力，如果需要突破这个限制，那么在客户端发送消息的时候，需要将消息按一定的时间进行聚合再上报，那么如何实现一个像以下需求的消息队列呢？

- [x] 没有消息时一直阻塞，避免CPU消耗
- [x] 一旦有消息的时候，只从消息队列里边取最多1秒的数据
- [x] 或者，一旦有足够消息数量的时候，立即返回

<!-- more -->

```go
package aggregate_queue

import (
	"time"
)

// 一种按时间聚合的Queue实现方式
type Queue struct {
	queue         chan interface{}
	bulkCount     int           // 按数量聚合
	aggregateTime time.Duration // 按时间聚合
}

func Create(queueLen int64, aggregate time.Duration, bulk int) *Queue {
	return &Queue{
		queue:         make(chan interface{}, queueLen),
		bulkCount:     bulk,
		aggregateTime: aggregate,
	}
}

func (q *Queue) Length() int {
	return len(q.queue)
}

func (q *Queue) Push(item interface{}) {
	q.queue <- item
}

func (q *Queue) Pops() (items []interface{}) {
	items = append(items, <-q.queue)
	t := time.After(q.aggregateTime)
	for {
		if len(items) >= q.bulkCount {
			return
		}
		select {
		case item := <-q.queue:
			items = append(items, item)
		case <-t:
			return
		}
	}
}

```

这里的实现方式，主要看`Pops`，如果没有消息，通过chan可以达到阻塞的目的（如果一直for循环读取slice的话，会导致CPU消耗非常之高），而一旦有了消息之后，立即创建一个`time.After`，然后通过`select`来继续从chan取数据，或者直到超时时间到达。

利用这种解决方式，我们实现了单机QPS超过两百万 ;-)