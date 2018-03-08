---
title: Flash system.img error
date: 2016-11-12 19:46:03
tags: [Tips]
---


使用fast boot烧写系统镜像的时候出现数据过大的错误：
```
sending 'system' (811488 KB)...
FAILED (remote: data too large)
finished. total time: 0.035s
```
烧入程序失败,Google了一下这个问题给出的解决是这样的：

<!-- more -->

```

**Analysis**
Fastboot IMG file defines the maximum size: 120MB
shaobin@shaobin-G41M-ES2L:~/cas/hs-android/main2.3$ find bootable/ -name '*.h' | xargs grep 'CFG_MAX_DOWNLOAD_BUF_LEN'bootable/bootloader/legacy/include/boot/config.h:#define CFG_MAX_DOWNLOAD_BUF_LEN (120*1024*1024)/* FIXME: 120MB */

**Solution**
Delete out/target/product/{x}/system/app directory of useless APK, and mkyaffs2image system.img
Increase CFG_MAX_DOWNLOAD_BUF_LEN size of macro, re- compile source code to generate fastboot.img, and refresh the fastboot

```
第一种方案亲测可行，但是前提是删除某些东西，这个很多情况下可能并不适用，而且打包的工具安装起来比较复杂(不过可以用来自己定制系统的时候使用)，第二种方案未亲自实验。

###solution
$fastboot flash -S 100M system system.img
s大写。
so easy