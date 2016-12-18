---
layout: post  
title: Android下内存修改的研究汇总
category: 移动开发  
tags: Android  安全  
keywords: Android  ptrace  内存修改
description:   
---
### 1.linux下有个开源的内存修改项目scanmem。

项目地址:

1.  [http://taviso.decsystem.org/scanmem.html](http://taviso.decsystem.org/scanmem.html "http://taviso.decsystem.org/scanmem.html")

2.  [http://code.google.com/p/scanmem/](http://code.google.com/p/scanmem/ "http://code.google.com/p/scanmem/")

3.  [https://github.com/coolwanglu/scanmem](https://github.com/coolwanglu/scanmem "https://github.com/coolwanglu/scanmem")

### 2.Android下要针对其他进程进行内存修改是基于ptrace系统调用，因此要想学会android下的进程注入，首先需要了解ptrace的用法。

* 参考博客：[http://blog.sina.com.cn/s/blog_4ac74e9a0100n7w1.html](http://blog.sina.com.cn/s/blog_4ac74e9a0100n7w1.html "http://blog.sina.com.cn/s/blog_4ac74e9a0100n7w1.html")

* 备注：在ubuntu下输入man ptrace命令，查看具体描述。

### 3.Android系统是基于Linux系统，在linux系统中可以通过ptrace系统调用实现对进程的内存修改。过程大致过程如下：

*   [1]通过远程进程pid，ATTACH到远程进程

*   [2]获取远程进程寄存器值，并保存，以便注入完成后恢复进程原有状态

*   [3]获取远程进程系统调用mmap、dlopen、dlsym调用地址

*   [4]与前端交互根据值搜索根据proc/&lt;pid&gt;/maps中的地址和偏移量

*   [5]根据值得变化，来确定内存地址（一般没办法精确到一个）

*   [6]根据内存地址修改成前端需要的值

*   [7]恢复远程进程寄存器

*   [8]DETACH远程进程。

备注：其中4-5两步可以有多种方式来搜索定位

参考：

[http://www.sanwho.com/133.html](http://www.sanwho.com/133.html "http://www.sanwho.com/133.html")

[http://blog.csdn.net/mldxs/article/details/14486827](http://blog.csdn.net/mldxs/article/details/14486827 "http://blog.csdn.net/mldxs/article/details/14486827")

[http://zhangwenxin82.blog.163.com/blog/static/114595956201171510512459/](http://zhangwenxin82.blog.163.com/blog/static/114595956201171510512459/ "http://zhangwenxin82.blog.163.com/blog/static/114595956201171510512459/")

---