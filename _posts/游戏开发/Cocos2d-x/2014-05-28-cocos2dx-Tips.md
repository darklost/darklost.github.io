---
layout: post  
title: Cocos2d-x Tips
category: 游戏开发  
tags: Cocos2d-x 
keywords: cocos2dx tips 
description:   
---


### 命名空间宏：
```
USING_NS_CC; // using namespace cocos2d

NS_CC_BEGIN； //using namespace cocos2d{

NS_CC_END; // }

```


* * *



### 判断一个精灵被点击：

1、层要接收点击消息。2、回调函数中取得点击坐标。3、取得精灵用boudingBox().containsPoint函数判断。（或使用 convertTouchToNodeSpaceAR 方法）



* * *



### 多Layer点击处理：

1、使用ccTouchesBegan()。此函数返回true，表示中断消息链，本层接收消息；返回false则本层不接收消息。

2、重写个Layer。大体思路是只有最底层的Layer接收消息，然后根据某种方式转发给各层。

具体可参考文章：[http://www.myexception.cn/operating-system/1118630.html](http://www.myexception.cn/operating-system/1118630.html) cocos2d-x 建立自己的层级窗口消息机制


### 精灵相关
* * *

精灵拉长：

setScale() 尽量不用这样的变换，因为会模糊。



* * *



精灵半透明：

setOpacity() 设置半透明0~255 。



* * *



精灵旋转：

setRotation() 默认是Z轴旋转。

setRotationX() X轴为对称轴旋转。

setRotationY() Y轴为对称中心。



* * *



精灵设定颜色：

setColor() 。



* * *

###动作相关

相反的动作：

reverse() 创建一个相反的动作，之前动作必须是By类型的。与坐标无关，只与动作相关。

相反一系列动作：

将CCSquence创建好的一系列动作赋值给一个CCFiniteTimeAction 指针，然后再调用这个指针的reverse。



* * *




动作类型：

CCActionInterval:








类名

功能




CCMoveTo

移动




CCScaleTo

放大




CCSKewTo

斜交（距离无穷的旋转）




CCRotateTo

旋转




CCJumpTo

跳动




CCBezierTo

贝塞尔曲线移动




CCBlink

闪烁




CCFadeIn\Out

渐隐




CCTintTo

上色




CCToggleVisibility

切换可见




CCHide

隐藏




CCShow

显示




CCOrbitCamera

轨道相机？能实现落叶翻转的效果




CCCardinalSplineBy

路径移动




CCCatmullRomTo

也是路径移动，不知道有什么区别










* * *



一直重复动作：

CCRepeatForever::create() 在runaAtion中把相应的动作套上这个类型即可。



* * *



重复一次动作：

CCRepeat::create() 在runaAtion中把相应的动作套上这个类型即可。



* * *



同步：

CCSpawn 与CCSquence用法一样只不过是同时执行。



* * *



跟随精灵移动：

CCFollow 运行Layer中的runAction。第二个参数为Layer的大小。



* * *



多个精灵的动作序列：

CCTargetedAction 与精灵相关的动作，创建好之后，可直接放到CCSqence中。



* * *



动作叠加：

精灵调用多次runAction可以使不同的动作叠加起来。



* * *



动作的暂停与恢复：


```C++
//动作暂停：
CCDirector::sharedDirector->getActionManager()->pauseAllRunningActions() /*即可暂停所有动作，返回值为一个CCSet* 要将其存入m_pPausedTargets中。使用时可参照：*/


 CC_SAFE_RELEASE(m_pPausedTargets);

m_pPausedTargets = director-getActionManager()-pauseAllRunningActions();

CC_SAFE_RETAIN(m_pPausedTargets);

//动作的恢复为：


sharedDirector-getActionManager()-resumeTargets(m_pPausedTargets)
```


---