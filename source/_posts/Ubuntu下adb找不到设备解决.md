---
title: Ubuntu下adb找不到设备解决
date: 2016-08-12 9:19:46
tags: [Linux,Ubuntu,Tips]
---


今天在编译了谷歌源码准备烧到手机里面的时候，发现在ubuntu 11.4下无法查找到设备。但是手机上可以识别usb连接。
这个时候我们要首先看一下这台手机是不是有开发者模式，有的话进入开发者模式中打开USB调试就可以发现我们的设备。
下面我们提供两种解决方案，当然前提是你的手机已经成功链接。

<!-- more -->

两种方案的前提都是查看usb连接的设备。
$lsusb
```
Bus 001 Device 002: ID 8087:8008 Intel Corp. 
Bus 002 Device 002: ID 8087:8000 Intel Corp. 
Bus 003 Device 002: ID 413c:2107 Dell Computer Corp. 
Bus 003 Device 003: ID 093a:2510 Pixart Imaging, Inc. Optical Mouse
Bus 003 Device 014: ID 0aaa:bbbb XXXX, Inc. 
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
Bus 002 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
Bus 003 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
Bus 004 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub

```

####方案一

将设备id添加到adb_usb.ini中。
在用户目录下，
$cd .android 
在该目录下找到adb_usb.ini文件，如果没有该文件的话，则创建。
$touch adb_usb.ini创建该文件。
$vim adb_usb.ini编辑该文件，添加一下内容
```
"# ANDROID 3RD PARTY USB VENDOR ID LIST -- DO NOT EDIT.
"# USE 'android update adb' TO GENERATE.
"# 1 USB VENDOR ID PER LINE.
0x0aaa
```
其中最后一行1234就是前面看到的设备id.



####方案二
1. $cd /etc/udev/rules.d找到51-android.rules
    $vim 51-android.rules
2. 在最后面加上
```SUBSYSTEM=="usb", ATTR{idVendor}=="0aaa", ATTR{idProduct}=="bbbb", MODE="0666", OWNER="<用户名>"
```
3. $sudo chmod a+rx /etc/udev/rules.d/51-android.rules    
    $sudo /etc/init.d/udev restart

####重启adb
$sudo adb kill-server
$sudo adb start-server
$sudo adb devices

DONE.