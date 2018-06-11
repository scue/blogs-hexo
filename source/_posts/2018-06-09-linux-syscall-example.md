---
layout: post
title: "Linux/Android系统调用示例"
description: ""
category: 技术
tags: [Linux, Syscall]
---

# 什么是系统调用

计算机系统的各种硬件资源是有限的，在现代多任务操作系统上同时运行的多个进程都需要访问这些资源，为了更好的管理这些资源进程是不允许直接操作的，所有对这些资源的访问都必须有操作系统控制。也就是说操作系统是使用这些资源的唯一入口，而这个入口就是操作系统提供的系统调用（System Call）。**在linux中系统调用是用户空间访问内核的唯一手段**，除异常和陷入外，他们是内核唯一的合法入口。

<!-- more -->

参考链接：https://blog.csdn.net/gatieme/article/details/50779184

# 调用案例

```c
#include <stdio.h>
#include <unistd.h>
#include <sys/syscall.h>
#include <sys/types.h>

#define SIZE (20)
int
main(int argc, char *argv[])
{
	pid_t tid;
	tid = syscall(SYS_gettid);
	printf("tid: %lu\n", tid);

	unsigned char buffer[SIZE] = "";
	syscall(SYS_getrandom, buffer, SIZE, 0);
	printf("random unsigned char: ");
	for(int i = 0; i < SIZE; ++i) printf("%02x", buffer[i]);
	printf("\n", buffer);
}
```

执行的结果：

```txt
root@dd1a6c93acc2:/tmp# gcc main.c -o main && ./main
tid: 184
random unsigned char: bdbdf292ac511bab766be7cc4002bca5e59150b4
root@dd1a6c93acc2:/tmp# vim main.c
root@dd1a6c93acc2:/tmp# ./main
tid: 186
random unsigned char: 71db26f6f34876a6ce8b652809a646c2d523b0bb
root@dd1a6c93acc2:/tmp# ./main
tid: 187
random unsigned char: b40bd7e3b365c7fbffdfef9bf60b73b1753d0c99
root@dd1a6c93acc2:/tmp#
```


# 其他信息

- 系统调用列表：`/usr/include/x86_64-linux-gnu/asm/unistd_64.h`，或`/usr/include/x86_64-linux-gnu/asm/unistd_32.h`
- `SYS_xxx`定义：`/usr/include/x86_64-linux-gnu/bits/syscall.h`


