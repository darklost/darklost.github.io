---
layout: post  
title: Cocos2d-x 2.x开发环境搭建（基于Eclipse）
category: 游戏开发  
tags: Cocos2d-x
keywords: cocos2dx tips 
description:   
---

**一需要的安装包**：

jdk ,adt-bundle, ndkr8+, python2.7, cocos2d-x-2.2.3;

**二，全局环境配置**

1.安装pythone,path 中添加如 C:\Python\;的安装路径 cmd python.exe 测试

2.解压NDK 3. cocos2d-x2.2.3 adt-bundle

3.打开Eclipse  指定NDK路径:windows- preferences-android-NDK;

4.添加cocos2d-x 源码link路径

5.添加c/c++环境变量

COCOS2DX = D:\devtool\cocos2d-x-2.2.3

NDK_ROOT= NDK解压路径

NDK_BUILD_WIN= ${NDK_ROOT}\ndk-build.cmd

NDK_MODULE_PATH= D:\devtool\cocos2d-x-2.2.3;D:\devtool\cocos2d-x-2.2.3\cocos2dx\platform\third_party\android\prebuilt;(注：可以用相对路径COCOS2DX;COCOS2DX \cocos2dx\platform\third_party\android\prebuilt;)

**三，创建项目**

静茹cocos2d-x 目录

cocos2d-x-2.2.3\tools\project-creator

shift+鼠标右键----在此处运行命令窗口

运行 create_project.py 根据提示创建 项目，

项目路径位于cocosd-x 目录的projects目录下

eclipse 导入android项目；

**四，项目环境配置**

右键导入的cocos2d-x 项目 propeties

C/C++ build  将build command 改为 ${NDK_BUILD_WIN}

打开C/C++编译器

查看include

将无法打开的在paths and Symbols 中将无法找到的在你当前的NDK和cocos2d中找到当前版本（不指定不影响编译，影响快捷提示和Eclipse报错）

**六，编译-****运行**

android.mk 可以自己编写，也可以使用提供的通用的android.mk(位于lib目录下)

需要时可以自己更改lib库的名字 （注意必须和java 中load 的名称对应）

C/C++编译器可以右键项目 build project /clean project

也可以在JAVA编译器 run android application project,编译后运行

c代码编译过程在 console CDT GLOBAL BUILD CONSOLE

小技巧：不影响编译的编译 其报错可以再如下途中选择忽略，建议改成warmming 避免没有代码错误提示

喜欢vs 的可以在新建的项目中win32.project文件夹中的sln 文件

---