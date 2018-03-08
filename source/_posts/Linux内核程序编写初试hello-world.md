---
title: Linux内核程序编写初试hello world
date: 2016-08-02 23:06:24
tags: [Linux]
---

本文为学习笔记，资料来自网络

##### 1  *.c文件的编写
$mkdir  test  创建实验目录，例如test,名字随意
$touch hello.c 创建hello.c文件
$vim hello.c 编写hello.c代码，主要完成打印“hello world”功能


hello.c 文件内容
```
//hello world driver for linux 

//包含必要的头文件，主要存在于linux内核源码中
#include <linux/kernel.h>  
#include <linux/init.h>  
#include <linux/module.h>  

//打印hello world的函数
static int __init hello_init(void)  
{  
    printk(KERN_ALERT "Hello, world! from kernel space....\n");  
    return 0;  
}  
 
//打印goodbye的函数
static void __exit hello_exit(void)  
{  
    printk(KERN_ALERT "Goodbye, world! Leaving kernel space....\n");  
}  

//module_init()指明模块的入口，这是必需的；
//module_exit()指明模块的出口，这也是必需的。
//传入的参数，上面的两个函数
module_init(hello_init);  
module_exit(hello_exit);  

```
##### 2 Makefile文件
为了方便使用make功能进行编译，make首先会扫面目录下的Makefile中的内容进行编译。
在与hello.c同一目录下创建一个Makefile文件，内容如下：
```
//编译目标对象，注意这里的后缀名是.o
obj-m += hello.o  
#pwd 当前所在目录
CURRENT_PATH:=$(shell pwd)  
#uname -r 查看当前的kernel版本
LINUX_KERNEL:=$(shell uname -r)  
#t生成linux内核的绝对路径
LINUX_KERNEL_PATH:=/usr/src/linux-headers-$(LINUX_KERNEL)  
#编译使用的命令
all:  
        //注意：make -C前面是一个TAB，否则会出现语法错误
	make -C $(LINUX_KERNEL_PATH) M=$(CURRENT_PATH) modules  
#clean  使用的命令
clean:  
	make -C $(LINUX_KERNEL_PATH) M=$(CURRENT_PATH) clean 
```
##### 3 编译
cd到.c文件所在目录，执行make命令。
执行ls查看生成的文件。成功的情况应该有一下文件，其中，hello.ko就是模块目标文件
```
hello.c   hello.mod.c  hello.o   modules.order
hello.ko  hello.mod.o  Makefile  Module.symvers
```
##### 4 加载到内核
$ sudo insmod hello.ko 将模块加载到内核中
此时，我们是看不到模块的输出的，需要查看日志
$dmesg 就能看到日志中输出的hello world，说明模块加载成功，如果日志过多，可使用$dmesg -c清除日志

##### 5 卸载模块
$ sudo rmmod hello.ko
$dmesg 就能看到日志中的goodbye,说明模块卸载成功

##### 6 常见问题
make时出现make: Nothing to be done for `all'.
解决此问题可以尝试一下两种方式：
- 可能是因为工程已经编译过，没发生改变，无需在编译，此时使用$make clean 清楚编译文件，重新进行编译
- Makefile中的TAB使用错误，对于这个错误在上面的Makefile中已经作出了说明，至于为什么会报这个错误，原因未知，有待探讨。