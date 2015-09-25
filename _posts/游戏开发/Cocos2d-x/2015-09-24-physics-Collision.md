---
layout: post  
title: Cocos2d-X 基于物理引擎不规则碰撞检测	
category: 游戏开发 	
tags: Cocos2d-X PhysicsEditor  
keywords: Cocos2d-X PhysicsEditor
description:   
---

#前言

```s
最近朋友问到关于不规则形状的碰撞检测问题，以前只是在某些书、Blog和帖子中看到过是通过物理引擎，没有自己实际写过代码测试，最近在找工作，闲暇时间也比较多，所以写了个测试验证下想法

```
项目地址：[github](https://github.com/darklost/PhysicTest01)

#1. 准备工作
	下载PhysicsEditor，用于编辑不规则图像[ps：按照我的理解就是拆分成多个三角形或者矩形]
	除了编辑形状用于拆分，还会有一些参数如摩擦力 恢复力 等的参数的设置
	如下面两图
![1](/public/img/cocos/physics/editor_01.png)
![2](/public/img/cocos/physics/editor_02.png)

#2. 编码测试
    如下图
![3](/public/img/cocos/physics/physicsTest.gif)


 






---








