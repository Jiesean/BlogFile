---
title: Android笔记之JNI机制二
date: 2016-09-05 23:10:43
tags: Android
---


>主要参考文献：
深入理解Android，卷１
Android 5.0 源码

相关文章：[Android笔记之JNI机制一](http://www.jianshu.com/p/d3b28b6ca56d)

学识尚浅，错误之处请指正。

有了前面文章的分析，我们了解了Android中JNI的基础的用法，这对我们继续对Android源码的纵向分析很有帮助。
接下来以Android 5.0中的wifi模块为例来分析JNI实现极其源码分布。

<!-- more -->

#####WifiNative.java
路径aidu：

存在静态代码段，加载动态库。

```
/* Register native functions */
    static {
        /* Native functions are defined in libwifi-service.so */
        System.loadLibrary("wifi-service");
        registerNatives();
  }
```
native关键字定义了很多的native函数，这里以注册函数为例。
```
private static native int registerNatives();
```