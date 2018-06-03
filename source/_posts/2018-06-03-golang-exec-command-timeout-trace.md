---
layout: post
title: "GO语言执行命令超时的设置"
description: ""
category: 技术
tags: [GO]
---

```go
package main
import (
    "bytes"
    "fmt"
    "log"
    "os/exec"
    "strings"
    "time"
    "syscall"
)
// 删除输出的\x00和多余的空格
func trimOutput(buffer bytes.Buffer) string {
    return strings.TrimSpace(string(bytes.TrimRight(buffer.Bytes(), "\x00")))
}
// 实时打印输出
func traceOutput(out *bytes.Buffer) {
    offset := 0
    t := time.NewTicker(time.Second)
    defer t.Stop()
    for {
        <-t.C
        result := bytes.TrimRight((*out).Bytes(), "\x00")
        size := len(result)
        rows := bytes.Split(bytes.TrimSpace(result), []byte{'\n'})
        nRows := len(rows)
        newRows := rows[offset:nRows]
        if result[size-1] != '\n' {
            newRows = rows[offset : nRows-1]
        }
        if len(newRows) < offset {
            continue
        }
        for _, row := range newRows {
            log.Println(string(row))
        }
        offset += len(newRows)
    }
}
// 运行Shell命令，设定超时时间（秒）
func ShellCmdTimeout(timeout int, cmd string, args ...string) (stdout, stderr string, e error) {
    if len(cmd) == 0 {
        e = fmt.Errorf("cannot run a empty command")
        return
    }
    var out, err bytes.Buffer
    command := exec.Command(cmd, args...)
    command.Stdout = &out
    command.Stderr = &err
    command.Start()
    // 启动routine等待结束
    done := make(chan error)
    go func() { done <- command.Wait() }()
    // 启动routine持续打印输出
    go traceOutput(&out)
    // 设定超时时间，并select它
    after := time.After(time.Duration(timeout) * time.Second)
    select {
    case <-after:
        command.Process.Signal(syscall.SIGINT)
        time.Sleep(time.Second)
        command.Process.Kill()
        log.Printf("运行命令（%s）超时，超时设定：%v 秒。",
            fmt.Sprintf(`%s %s`, cmd, strings.Join(args, " ")), timeout)
    case <-done:
    }
    stdout = trimOutput(out)
    stderr = trimOutput(err)
    return
}
func main() {
    stdout, stderr, e := ShellCmdTimeout(5, "ping", "114.114.114.114")
    log.Println("stdout:", stdout)
    log.Println("stderr:", stderr)
    log.Println(e)
}
```

值得注意的是应当先发送`SIGINT`再去KILL掉命令，这样子可以确保它的输出完整。

比如，一般的直接Kill掉的程序的输出：

```txt
2018/05/30 21:08:00 PING 114.114.114.114 (114.114.114.114): 56 data bytes
2018/05/30 21:08:00 64 bytes from 114.114.114.114: icmp_seq=0 ttl=86 time=32.105 ms
2018/05/30 21:08:02 64 bytes from 114.114.114.114: icmp_seq=1 ttl=82 time=30.083 ms
2018/05/30 21:08:02 64 bytes from 114.114.114.114: icmp_seq=2 ttl=79 time=31.818 ms
2018/05/30 21:08:04 运行命令（ping 114.114.114.114）超时，超时设定：5 秒。
2018/05/30 21:08:04 stdout: PING 114.114.114.114 (114.114.114.114): 56 data bytes
64 bytes from 114.114.114.114: icmp_seq=0 ttl=86 time=32.105 ms
64 bytes from 114.114.114.114: icmp_seq=1 ttl=82 time=30.083 ms
64 bytes from 114.114.114.114: icmp_seq=2 ttl=79 time=31.818 ms
64 bytes from 114.114.114.114: icmp_seq=3 ttl=64 time=40.516 ms
64 bytes from 114.114.114.114: icmp_seq=4 ttl=60 time=29.796 ms
2018/05/30 21:08:04 stderr: 
2018/05/30 21:08:04 <nil>

```

而，现实中，你可能期望得到的是：

```txt
2018/05/30 21:17:58 PING 114.114.114.114 (114.114.114.114): 56 data bytes
2018/05/30 21:17:58 64 bytes from 114.114.114.114: icmp_seq=0 ttl=74 time=31.938 ms
2018/05/30 21:18:00 64 bytes from 114.114.114.114: icmp_seq=1 ttl=70 time=33.120 ms
2018/05/30 21:18:00 64 bytes from 114.114.114.114: icmp_seq=2 ttl=70 time=30.973 ms
2018/05/30 21:18:03 64 bytes from 114.114.114.114: icmp_seq=3 ttl=59 time=30.543 ms
2018/05/30 21:18:03 64 bytes from 114.114.114.114: icmp_seq=4 ttl=79 time=33.236 ms
2018/05/30 21:18:03 
2018/05/30 21:18:03 --- 114.114.114.114 ping statistics ---
2018/05/30 21:18:03 5 packets transmitted, 5 packets received, 0.0% packet loss
2018/05/30 21:18:03 round-trip min/avg/max/stddev = 30.543/31.962/33.236/1.091 ms
2018/05/30 21:18:03 运行命令（ping 114.114.114.114）超时，超时设定：5 秒。
2018/05/30 21:18:03 stdout: PING 114.114.114.114 (114.114.114.114): 56 data bytes
64 bytes from 114.114.114.114: icmp_seq=0 ttl=74 time=31.938 ms
64 bytes from 114.114.114.114: icmp_seq=1 ttl=70 time=33.120 ms
64 bytes from 114.114.114.114: icmp_seq=2 ttl=70 time=30.973 ms
64 bytes from 114.114.114.114: icmp_seq=3 ttl=59 time=30.543 ms
64 bytes from 114.114.114.114: icmp_seq=4 ttl=79 time=33.236 ms

--- 114.114.114.114 ping statistics ---
5 packets transmitted, 5 packets received, 0.0% packet loss
round-trip min/avg/max/stddev = 30.543/31.962/33.236/1.091 ms
2018/05/30 21:18:03 stderr: 
2018/05/30 21:18:03 <nil>

```

看到不同了吗？接收到SIGINT之后，PING还有很重要的一些信息输出

```txt
--- 114.114.114.114 ping statistics ---
5 packets transmitted, 5 packets received, 0.0% packet loss
round-trip min/avg/max/stddev = 30.543/31.962/33.236/1.091 ms

```

此外，为了让`SIGINT`信号可以被正常接收，请不要使用`sh -c 命令`来执行。

> 提示：看到`trimOutput`函数了吗，它的作用就像是`tail -f`命令一样，可以实时跟踪输出哟~



