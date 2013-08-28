---
title: php进程的内存使用检测
layout: post
tags: php 优化 系统
category: 优化
comments: true
---
{% include JB/setup %}

如何比较快速的检测线上php进程的内存使用情况，找出那些内存消耗“大户”，昨天有个朋友问我这个问题，我想是否可以使用strace跟踪进程的系统调用来观察内存申请的情况。

我们知道，内存的申请是通过brk(2)和mmap(2)这两个系统调用完成的，可以通过观察它们的调用情况来确定谁申请大块的内存，我在线上一个服务器上测试了一下，过程如下：

    strace -p 30264 -t -v -s 200 -o st.log
    30264是一个php fpm的进程id；t是打印绝对时间；v是可视模式；s是输出最大长度；输出到st.log

大概收集了5分钟的数据，有5mb大小，分析结果，按照申请的内存大小排序：

    cat st.log | grep mmap | awk '{print $3}' | sort -n

brk也查找了一下，但是没有结果，mmap倒是很多，最大的是70276字节，是个php公共类库，有69kb。

通过这种方法应该也可以查到从mysql读取大量数据的情况，但是具体没有试过。



