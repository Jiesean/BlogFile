---
title: Linux基础之gcc
date: 2016-12-08 22:43:57
tags: [Linux]
---


### 前言
在Linux开发环境中，比较常用的C\C++的编译器当属gcc（GNU Compiler Collection）了,当然它能够编译的不止C\C++,它不仅能够编译多种语言，而且还是一个交叉编译的平台编译器，所以很有必要好好学习一下的。

### 安装
在ubuntu 12.4下，gcc是默认安装的，如果系统未安装gcc，安装也是很简单的：
```
$sudo apt-get install gcc
```

<!-- more -->

还需要安装常用的头文件和库文件：
```
$sudo apt-get install build-essential
```
完成，可以通过使用一下gcc -v查看gcc版本。
```
$gcc -v
Target: x86_64-linux-gnu
Configured with: ../src/configure -v --with-pkgversion='Ubuntu/Linaro 4.6.3-1ubuntu5' --with-bugurl=file:///usr/share/doc/gcc-4.6/README.Bugs --enable-languages=c,c++,fortran,objc,obj-c++ --prefix=/usr --program-suffix=-4.6 --enable-shared --enable-linker-build-id --with-system-zlib --libexecdir=/usr/lib --without-included-gettext --enable-threads=posix --with-gxx-include-dir=/usr/include/c++/4.6 --libdir=/usr/lib --enable-nls --with-sysroot=/ --enable-clocale=gnu --enable-libstdcxx-debug --enable-libstdcxx-time=yes --enable-gnu-unique-object --enable-plugin --enable-objc-gc --disable-werror --with-arch-32=i686 --with-tune=generic --enable-checking=release --build=x86_64-linux-gnu --host=x86_64-linux-gnu --target=x86_64-linux-gnu
Thread model: posix
gcc version 4.6.3 (Ubuntu/Linaro 4.6.3-1ubuntu5) 
```
### 编译简介
gcc的编译程序的分为四个阶段：
- 预处理（pre-processing）
- 编译（compiling）
- 汇编（assembing）
- 连接（linking）

因为在大多数情况下，我们并不需要中间过程，所以四个过程直接整合成一个，直接得到的是可执行程序。

以一个简单的c语言程序为例说明：
1. 创建c程序,来个最简单的hello world
```
#include <stdio.h>
int main(void){
   printf("Hello World!");
   return 0;
}
```
2. 编译
```
$gcc hello.c -o hello
```
编译生成hello这个可执行程序。
3. 执行
```
$./hello
```
得到输出结果Hello world!

### 编译分解
1. 预处理
预处理阶段负责处理预处理命令，例如其中的#include、#define等。
使用-E可以使编译停在预处理结果：
```
$gcc -E hello.c -o hello.i
```
查看生成的hello.i会发现stdio.h被实际的文件内容所替代，宏定义等预处理也被实际内容所替代。
2. 编译
编译阶段是将预处理完成的程序编译成汇编语言。
```
$gcc -S hello.i -o hello.s
```
查看生成的hello.s,发现已经变成汇编语言：
```
        .file   "hello.c"
        .section        .rodata
       .LC0:
        .string "Hello World!"
        .text
        .globl  main
        .type   main, @function
       main:
       .LFB0:
        .cfi_startproc
        pushq   %rbp
        .cfi_def_cfa_offset 16
        .cfi_offset 6, -16
        movq    %rsp, %rbp
        .cfi_def_cfa_register 6
        movl    $.LC0, %eax
        movq    %rax, %rdi
        movl    $0, %eax
        call    printf
        movl    $0, %eax
        popq    %rbp
        .cfi_def_cfa 7, 8
        ret
        .cfi_endproc
       .LFE0:
        .size   main, .-main
        .ident  "GCC: (Ubuntu/Linaro 4.6.3-1ubuntu5) 4.6.3"
        .section        .note.GNU-stack,"",@progbits
```
3. 汇编
在汇编阶段，gcc把汇编的代码编译成CPU可以执行的机器码，也就是目标代码模块。
```
$gcc -c hello.s -o hello.o
```
4. 连接
连接截断把生成的目标模块以及各种库文件连接在一起，形成一个二进制的可执行文件。
```
$gcc hello.o -o hello
```
得到的可执行文件就可以运行了。

### 多目标编译
通常情况下，为了实现模块化编程的目的，一个程序通常会被分成若干个源文件，这时当然不需要每一个文件都编译一遍再连接到一起，gcc会自动帮你完成这些事情。
```
$gcc file1.c file2.c file3.c -o hello 
```
当然它其实相当于每个分别编译然后连接了：
```
$gcc file1.c -o file1.o
$gcc file2.c -o file2.o
$gcc file3.c -o file3.o
$gcc file1.o file2.o file3.o  -o hello
```
当然更复杂的情况可能需要使用make这个工具去完成了。