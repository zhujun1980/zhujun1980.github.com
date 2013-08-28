---
title: httpfollow
layout: post
tags: 工具 系统
category: 工具
comments: true
---
{% include JB/setup %}

线上服务器的日志是定时汇总的，有时排查错误常常需要查看最新的记录，比如错误日志，之前一直使用curl从命令行直接获取，但是当文件很大的时候要等很久才能看到最后的内容，周末在家做了个小工具，[httpfollow](https://github.com/zhujun1980/httpfollow)，使用http range来跟踪最新的内容，有点类似`tail -f`

####使用方法

    httpfollow http://www.example.com -c 1024

    httpfollow http://www.example.com/file -c 2000 -f

    httpfollow http://user:pass@www.example.com:8080/path/to/file.log -f

curl也支持range，但是需要你自己记住访问到那里了，httpfollow可以帮你记住。
