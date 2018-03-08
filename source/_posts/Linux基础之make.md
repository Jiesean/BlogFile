---
title: Linux基础之make
date: 2016-12-26 20:14:53
tags: [Linux]
---


### 简介
上一篇文章中提到的gcc可以对源文件进行编译，gcc可以同时编译一个或者多个源文件并把他们链接起来组成可执行的二进制文件，但是如果涉及到大型的工程时，直接gcc可能会非常的复杂，而且部分文件的修改可能造成重复编译的浪费。
而make就是linux下的一个负责完成该工作的自动编译管理器。最基本的原理就是当执行make命令时，先去当前文件夹中查找名称为**Makefile**，找不到再去寻找**makefile**
的文件，然后按照该文件里面制定的编译规则去编译整个工程。

<!-- more -->

### Makefile格式

以简单的输出hello world的c语言程序的编译进行说明：

test.c文件内容如下：
```
#include <stdio.h>

int main(){
	printf("Hello world\n");
	return 0;
	}
```
编写一个简单的Makefile文件：
```
#Hello world makefile

hello:test.o
	gcc  test.o -o hello

test.o:test.c
	gcc -c  test.c -o test.o
	
all:test.o test.c
	gcc -c  test.c -o test.o
	
clean:
	rm hello
	
cleanall:
	rm test.o hello
	
```
对该Makefile文件进行说明：
```
#Hello world makefile
```
是注释行。
除了注释以外，Makefile文件由一条条的动作（我自己起的名字）组成，这种动作的标准定义是：
```
<target> : <prerequisites> 
[tab] <commands>
```
target相当于该条动作的名称，在linux下，直接
```
$make target
```
即可执行该动作。
- 注：后面不跟target的情况下，默认执行Makefile第一条命令。

prerequisites名为前置条件，也就是依赖，就是执行该动作所依赖的文件列表，commands为命令，这里试标准的linux shell命令行，行数没有限定。

那么我们执行一条动作时，该动作到底做了什么呢？
- Step1: 该动作没有依赖文件，直接执行命令。
- Step2: 该动作提供了依赖文件列表，而且依赖文件都存在，且没有更新（依赖文件的last-modification时间戳都比目标的时间戳早，也就是意味着，上次执行该动作后，所依赖的文件都没有被修改），直接执行命令行。
- Step3: 该动作的依赖文件不存在或者文件被更新过，那么将已该依赖文件为目标，来寻找构建该目标的动作，新建或者重新生成该依赖文件。

下面以上面的Makefile文件为例进行说明：
1. $ make hello
该动作依赖test.o这个文件，发现该文件并不存在，因此执行Step3,查找构建test.o的动作并执行，创建成功后，在执行hello动作中的命令行，因此该动作的执行结果是：
```
gcc -c  test.c -o test.o
gcc  test.o -o hello
```
生成了test.o和hello。执行hello输出Hello world。
2. $make clean
该动作没有依赖文件，所以直接执行命令行：
```
rm hello
```
将刚刚生成的hello这个可执行文件删除。
3. 再次执行$make hello
因为依赖文件已经存在，而且没有经过更新，所以直接执行命令行：
```
gcc  test.o -o hello
```
生成hello。

### 小结
Makefile不仅是简单的执行shell命令，他还能很好的支持很多的语法，比如宏定义，字符匹配，变量，函数，循环和判断等，这些语法能够帮助我们很好的控制大型工程的编译工作，避免重复编译。