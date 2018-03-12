---
layout: post
date: 2015-05-10 00:00:00
title: "Android抓取所有日志"
description: ""
category: 技术
tags: Android
---

总有些时候我们会遇到非必现场景，若未及时抓取日志，可能为时已晚

### 抓取日志

1.  抓取Logcat日志

    这个是最简单的，使用-f重定向到一个文本文件即可，`-v threadtime`则显示详细时间和进程

        # for logcat
        {
                log -t logcat_all "start logcat to file: $logcat_file"
                    /system/bin/logcat -v threadtime -f $logcat_file
        }   &

2.  抓取Kmsg日志

    通过抓取`/proc/kmsg`来完成

        # for kmsg
        {
                log -t logcat_all "start kmsg to file: $kmsg_logfile"
                    cat /proc/kmsg >$kmsg_logfile
        }   &

3.  抓取Top日志
    
    通过抓取Top可以看到系统运行时，占用CPU的情况等等

        # for top
        {
            while busybox true; do
                top_newtime=$(/system/bin/date +%F_%H-%M-%S)
                top_logfile=$logcat_dir/top_${top_newtime}.log && > $top_logfile
                log -t logcat_all "start top to file: $top_logfile"
                # 仅保留5个top日志文件
                # 5*100*3=25分钟内的Top信息，避免Top日志文件过大
                busybox rm -f $(busybox ls -1t ${logcat_dir}/top_* | busybox tail -n +6)
                for n in $(busybox seq 1 100) ; do
                    # 每3秒打印一次top信息，并加入时间显示
                    top -m 5 -d 3 -t -n 1 | busybox awk '{now=strftime("%Y-%M-%d %T "); print now $0}' >>$top_logfile
                    echo >>$top_logfile
                done
            done
        }   &

    为了方便分析结果，这里添加了详细时间，同时限定了Top输出大小（避免SD卡占用过快）

4.  抓取vmstat日志

    通过抓取vmstat日志，可分析的系统信息就更多了

        # for vmstat
        {
            while busybox true; do
                log -t logcat_all "start vmstat_ to file: $vmstat_logfile"
                vmstat_newtime=$(/system/bin/date +%F_%H-%M-%S)
                vmstat_logfile=$logcat_dir/vmstat_${vmstat_newtime}.log && > $vmstat_logfile
                busybox rm -f $(busybox ls -1t ${logcat_dir}/vmstat_* | busybox tail -n +6)
                /system/bin/vmstat -d 3 -n 100 | busybox awk '{now=strftime("%Y-%M-%d %T "); print now $0}' >$vmstat_logfile 2>&1
            done
        }   &

    同时也添加了详细的时间输出，方便分析

### 开机运行

在`init.rc`添加一个Service，当系统已正常启动完成时，执行抓取日志

    service logcat_all /system/bin/logcat_wrapper.sh
            class main
            disabled
            oneshot

    on property:dev.bootcomplete=1
        start logcat_all

其中，`logcat_wrapper.sh`内容如下：

    #!/system/bin/sh
    {
            /system/bin/busybox nohup /system/bin/logcat_all.sh > /dev/null 2>&1 &
    }   &

使用nohup执行，避免脚本被杀死，而`logcat_all.sh`则见此链接 [logcat_all.sh](https://gist.github.com/scue/fbbfdf9e09fe805ff041)
