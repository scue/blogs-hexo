---
layout: post
title: "谈谈我工作中的服务器后端的性能压测"
description: ""
category: 技术
tags: [benchmark, wrk]
---

前段时间写了一个升级查询的后台服务器，涉及到服务器的性能测试，这里仅作记录：

<!-- more -->

# 单个测试

首先，我先来验证服务器的准确性，通过curl请求来确保数据是OK的：

> 提示：所有的数据传输都是https，若请求的是http也会重定向到https

单个测试http转发:

```sh
curl -H "Content-Type: application/json" http://update.xxx.com/checkversion -L --post301 -d '
{
// POST内容保密
}'
# CURL 添加 `-L --post301` 可以进行POST重定向http至https
```

单个测试https:

```sh
curl -H "Content-Type: application/json" https://update.xxx.com/checkversion -d '
{
// POST内容保密
}'
```

# 并发测试

## ab

```sh
$ ab -c 1000 -n 10000 -T "application/json" -p /tmp/xxx.json https://update.xxx.com/checkversion
```

ab是apachebench命令的缩写。

ab是apache自带的压力测试工具。ab非常实用，它不仅可以对apache服务器进行网站访问压力测试，也可以对或其它类型的服务器进行压力测试。

参考链接：https://www.cnblogs.com/myvic/p/7703973.html

## jmeter:

```sh
brew install jmeter --with-plugins
jmeter -t /Users/scue/docs/xxx/发布系统/test-weiqiang.jmx
```

Jmeter有一个界面进行测试的配置，但个人在实际使用过程中发现它对测试机的性能要求较高~

参考链接：http://www.cnblogs.com/TankXiao/p/4045439.html

## wrk ✔️

* 环境安装

```sh
sudo yum groupinstall 'Development Tools'
sudo yum install openssl-devel
sudo yum install git
git clone https://github.com/wg/wrk.git wrk
cd wrk
make -j8
cp wrk /usr/local/bin/
```

* 执行测试

```sh
$ wrk -c 3000 -d 10 -t 1000 -s scripts/post_json.lua https://update.xxx.com/checkversion

$ cat scripts/post_json.lua
wrk.method = "POST"
wrk.body   = [[{
-- POST内容保密
}]]
wrk.headers["Content-Type"] = "application/json"
-- function response(status, headers, body)  
--    print(body)
-- end  
```

* 并发结果

```txt
Running 10s test @ https://update.xxx.com/checkversion
  1000 threads and 3000 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   297.84ms   82.24ms   1.08s    81.53%
    Req/Sec    11.12      5.91    60.00     61.59%
  129844 requests in 10.14s, 80.98MB read
Requests/sec:  12803.26
Transfer/sec:      7.99MB
```

表示每秒请求数量达到`12803.26`个。

参考链接：https://type.so/linux/lua-script-in-wrk.html

# 一些建议

以上几个测试方法，我比较推荐使用wrk，配置比较灵活，各方面还都很OK，同时有几点建议：

- 测试机要与服务器网络足够稳定且宽带足够（尽量同一个LAN）
- 测试程序不要放到服务器上同时跑
- 可以使用多个机器对同一台服务器进行测试（考虑测试机宽带、性能消耗）
- 增加测试机不能提高测试的结果时，这个结果应该是我们想要测试结果了

![](https://ws1.sinaimg.cn/large/6e22ca27gy1fs08fj6hopj20n30jzn1y)

> 提示：使用iTerm2的`Broadcast input to all panes in current tab` （Opt+Cmd+I）同时多个Pane进行输入文字很好用，可以同时开始测试。

