---
layout: post  
title: Cocos2d-x Android NDK BUG调试方式	
category: 移动开发  
tags: Android Cocos2d-x
keywords: Android NDK debug
description:   
---
***常见的错误捕获方法***

###1 logcat
Android最常用的调试方式`logcat`:

常见的问题都会在logcat中显示

*	优势：java层的错误很容易看出来
*	劣势：c/c++层的问题很难看出来

###2 ndk-stack

Android C/C++代码出问题比较好用的方式


*	优势：常见C/C++代码的问题能通过崩溃的堆栈信息查看
*	劣势：内存问题很难检测出来

###3 ndk-gdb

`注意`

>导出环境变量：
>
>`ANDROID_NDK_ROOT=/user/dengke/android/android-ndk-r10c`
>`export ANDROID_NDK_ROOT`
>
>编译：
>
>`ndk-build clean all NDK_DEBUG=1`
>
>终端运行：
>`ndk-gdb --start`(如果程序app已运行直接输入 `ndk-gdb`)
>
>运行app,然后终端执行`gdb命令`

*	优势：可以断点debug，对比较麻烦的内存问题效果不错
*	劣势：调试比较麻烦


	
---