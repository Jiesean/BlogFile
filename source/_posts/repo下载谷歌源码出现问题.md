---
title: repo下载谷歌源码出现问题
date: 2016-08-09 9:10:19
tags: [Tips,Android]
---

使用repo下载谷歌源码时，出现了如下错误。


执行   $repo init -u git远程库地址 -b git分支 命令时出现：

```
error: in sync: [Errno 2] No such file or directory: u'/home/ubuntu/workspace/packages/apps/VoiceDialer/.git/HEAD' 
error: manifest missing or unreadable -- please run init
```

解决办法：
将文件夹删除，其父文件夹中的 .repo文件夹也删除。
创新建立文件夹，再执行$repo init

错误消失