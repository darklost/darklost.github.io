---
layout: post
title: Linux 0：实际问题
category: 技术
tags: Linux
keywords: 
description: 
---
大学基本上没用过Linux和Unix(Mac os)系列的操作系统，工作后从事Android和手游开发，Android和iOS就是基于这两个操作系统的，那且得学着吧

- [《鸟哥的Linux私房菜——基础学习篇》](http://book.douban.com/subject/4889838/)
- [《鸟哥的Linux私房菜——服务器架设篇》](http://book.douban.com/subject/2338464/)

本系列文章分为两部分：

- 系统学习上面两本书的笔记。
- 实际中遇到的问题及解决方案，即本文内容。


##实际问题
##1. 建立网络映射
- Mac：`Finder`->`前往`->`连接服务器`->输入`smb://IPaddress/samba`->`连接`
- Linux：`位置`->`连接服务器`->“服务类型”选择`自定义位置`->输入`smb://IPaddress/samba`->`连接`

##2. ssh登陆失败
以root身份远程登陆服务器，密码正确，却显示如下警告：

```bash
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that a host key has just been changed.
The fingerprint for the RSA key sent by the remote host is
3a:17:4b:6e:62:e6:94:df:09:78:99:90:51:68:18:62.
Please contact your system administrator.
Add correct host key in /Users/AnyaLin/.ssh/known_hosts to get rid of this message.
Offending RSA key in /Users/AnyaLin/.ssh/known_hosts:4
RSA host key for 222.195.93.129 has changed and you have requested strict checking.
Host key verification failed.
```

解决方法：

```bash
vi ~/.ssh/known_hosts     #选中最后一条登陆记录，双击`d`删除，按“：”进入末行编辑模式，输入“x”，回车
ssh root@222.195.93.129   #再次登陆
The authenticity of host '222.195.93.129 (222.195.93.129)' can't be established.
RSA key fingerprint is 3a:17:4b:6e:62:e6:94:df:09:78:99:90:51:68:18:62.
Are you sure you want to continue connecting (yes/no)?    #输入yes
```

##3. tar.gz 文件解压
- 打开终端
- 进入需要解压的xxxx.tar.gz文件所在目录
- $ tar xvfz xxxx.tar.gz

<<<<<<< HEAD
##4. 新建文件命令

```
touch a.txt
```
=======
>>>>>>> cc60f2ed32a5f9727a68a2eb7f2f6a8a1eeac51a



