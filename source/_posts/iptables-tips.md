---
layout: post
date: 2015-05-21 00:00:00
title: "Iptables使用小技巧"
description: ""
category: 技术
tags: 
  - iptables
---

### 1. 转发本地端口

转发本地端口到指定端口，如转发80端口到2000

    sudo iptables -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-ports 2000

### 2. 转发本地端口至远程端口

这个我们可以简单理解为端口映射，路由器经常这么干

    # 200.200.137.235:52345 <==> 200.200.137.172:12345
    sudo iptables -t nat -A PREROUTING -p tcp --dport 52345 -j DNAT --to-destination 200.200.137.172:12345
    sudo iptables -t nat -A POSTROUTING -p tcp --dport 12345 -j MASQUERADE

当然，还要以使用ssh来实现这个功能：

    # 操作PC: 200.200.137.235
    autossh -NfL  0.0.0.0:52345:200.200.137.172:12345 lwq@localhost

    # 操作PC： 200.200.137.172
    autossh -NfR  0.0.0.0:52345:localhost:12345 lwq@200.200.137.235

### 3. 作为网关

比如我期望 200.200.88.70 能够作为网关让大家可以畅游互联网

    sysctl net.ipv4.ip_forward=1
    iptables -t nat -A POSTROUTING -s 200.200.88.0/22 -o eth0 -j MASQUERADE
    iptables -t nat -A POSTROUTING -o tun0 -j MASQUERADE # tun0是ssh_vpn打开的网口

### 4. Ubuntu WiFi热点

360 WiFi共享热点方法: [360-wifi-linux](https://github.com/yajin/360-wifi-linux)

    sudo iptables -t nat -A POSTROUTING -s ${subnet}.0/24 -o $out_interface -j MASQUERADE
    sudo iptables -A FORWARD -s ${subnet}.0/24 -o $out_interface -j ACCEPT
    sudo iptables -A FORWARD -d ${subnet}.0/24 -m conntrack --ctstate ESTABLISHED,RELATED -i $out_interface -j ACCEPT

### 5. 限制QQ上网

    #星期一到星期六的8:00-12:30禁止qq通信 
    iptables -I FORWARD -p udp --dport 53 -m string --string "tencent" -m time --timestart 8:15 --timestop 12:30 --days Mon,Tue,Wed,Thu,Fri,Sat -j DROP 
    iptables -I FORWARD -p udp --dport 53 -m string --string "TENCENT" -m time --timestart 8:15 --timestop 12:30 --days Mon,Tue,Wed,Thu,Fri,Sat -j DROP 
    #星期一到星期六的8:00-12:30禁止qq通信 
    iptables -I FORWARD -p udp --dport 53 -m string --string "tencent" -m time --timestart 13:30 --timestop 20:30 --days Mon,Tue,Wed,Thu,Fri,Sat -j DROP 
    iptables -I FORWARD -p udp --dport 53 -m string --string "TENCENT" -m time --timestart 13:30 --timestop 20:30 --days Mon,Tue,Wed,Thu,Fri,Sat -j DROP 
    #星期一到星期六的8:00-12:30禁止qq网页 
    iptables -I FORWARD -s 192.168.0.0/24 -m string --string "qq.com" -m time --timestart 8:15 --timestop 12:30 --days Mon,Tue,Wed,Thu,Fri,Sat -j DROP 
    iptables -I FORWARD -s 192.168.0.0/24 -m string --string "qq.com" -m time --timestart 13:00 --timestop 20:30 --days Mon,Tue,Wed,Thu,Fri,Sat -j DROP 

### 6. 限制指定字符串

    #禁止ay2000.net，宽频影院，色情，广告网页连接 ！但中文 不是很理想
    iptables -I FORWARD -s 192.168.0.0/24 -m string --string "ay2000.net" -j DROP 
    iptables -I FORWARD -d 192.168.0.0/24 -m string --string "宽频影院" -j DROP 
    iptables -I FORWARD -s 192.168.0.0/24 -m string --string "色情" -j DROP 
    iptables -I FORWARD -p tcp --sport 80 -m string --string "广告" -j DROP 

    #可以通过 hex 来限制，因为编码是很多种的，通常UTF和GB2312做一下限制
    iptables -A FORWARD -p udp --dport 53 -m string --hex-string "|00 00 ff 00 01|" --to 255 --algo bm -m comment --comment "IN ANY?" -j DROP

### 7. 限制流量

    iptables -A INPUT -p icmp -m limit --limit 3/s
