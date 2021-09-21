---
layout: post  
title: Cocos2d-x 场景动画切换过渡
category: 游戏开发  
tags: Cocos2d-x 
keywords: cocos2dx scene 
description:   
---
```C++
void StartScene::beginGame()

{

CCLog("beginGame");

//CCTransitionScene *trans = CCTransitionScene::create(2, level);

//CCDirector::sharedDirector()-&gt;replaceScene(level);

//场景切换

    CCTransitionScene *reScene = NULL;

    CCScene *s = LevelScene::scene();

    float t = 1.2f;

// CCTransitionJumpZoom

// 作用： 创建一个跳动的过渡动画

// 参数1：过渡动作的时间

// 参数2：切换到目标场景的对象

    reScene = CCTransitionJumpZoom ::create(t , s);

    CCDirector::sharedDirector()-&gt;replaceScene(reScene);

// CCTransitionProgressRadialCCW

// 作用： 创建一个扇形条形式的过渡动画， 逆时针方向

// 参数1：过渡动作的时间

// 参数2：切换到目标场景的对象

    reScene = CCTransitionProgressRadialCCW::create(t, s);

    CCDirector::sharedDirector()-&gt;replaceScene(reScene);

    CCTransitionProgressRadialCW

// 作用： 创建一个扇形条形式的过渡动画， 顺时针方向

// 参数1：过渡动作的时间

// 参数2：切换到目标场景的对象

    reScene = CCTransitionProgressRadialCW::create(t,s);

    CCDirector::sharedDirector()-&gt;replaceScene(reScene);

    CCTransitionProgressHorizontal

// 作用： 创建一个水平条形式的过渡动画，

// 参数1：过渡动作的时间

// 参数2：切换到目标场景的对象

    reScene = CCTransitionProgressHorizontal ::create(t,s);

    CCDirector::sharedDirector()-&gt;replaceScene(reScene);

// CCTransitionProgressVertical

// 作用： 创建一个垂直条形式的过渡动画，

// 参数1：过渡动作的时间

// 参数2：切换到目标场景的对象

    reScene = CCTransitionProgressVertical::create(t, s);

    CCDirector::sharedDirector()-&gt;replaceScene(reScene);

// CCTransitionProgressInOut

// 作用： 创建一个由里向外扩展的过渡动画，

// 参数1：过渡动作的时间

// 参数2：切换到目标场景的对象

    reScene = CCTransitionProgressInOut::create(t, s);

    CCDirector::sharedDirector()-&gt;replaceScene(reScene);

// CCTransitionProgressOutIn

// 作用： 创建一个由外向里扩展的过渡动画，

// 参数1：过渡动作的时间

// 参数2：切换到目标场景的对象

    reScene = CCTransitionProgressOutIn::create(t, s);

    CCDirector::sharedDirector()-&gt;replaceScene(reScene);

// CCTransitionCrossFade

// 作用：创建一个逐渐透明的过渡动画

// 参数1：过渡动作的时间

// 参数2：切换到目标场景的对象

    reScene = CCTransitionCrossFade::create(t, s);

    CCDirector::sharedDirector()-&gt;replaceScene(reScene);

// CCTransitionPageTurn

// 作用：创建一个翻页的过渡动画

// 参数1：过渡动作持续的时间

// 参数2：切换的目标场景的对象

// 参数3：是否逆向翻页

    reScene = CCTransitionPageTurn::create(t, s, false);

    CCDirector::sharedDirector()-&gt;replaceScene(reScene);

// CCTransitionFadeTR

// 作用：创建一个部落格过渡动画， 从左下到右上

// 参数1：过渡动作持续的时间

// 参数2：切换的目标场景的对象

    reScene =CCTransitionFadeTR::create(t, s);

    CCDirector::sharedDirector()-&gt;replaceScene(reScene);

// CCTransitionFadeBL

// 作用：创建一个部落格过渡动画， 从右上到左下

// 参数1：过渡动作持续的时间

// 参数2：切换的目标场景的对象

    reScene = CCTransitionFadeBL::create(t, s);

    CCDirector::sharedDirector()-&gt;replaceScene(reScene);

// CCTransitionFadeUp

// 作用：创建一个从下到上，条形折叠的过渡动画

// 参数1：过渡动作持续的时间

// 参数2：切换的目标场景的对象

    reScene= CCTransitionFadeUp::create(t, s);

    CCDirector::sharedDirector()-&gt;replaceScene(s);

// CCTransitionFadeDown

// 作用：创建一个从上到下，条形折叠的过渡动画

// 参数1：过渡动作持续的时间

// 参数2：切换的目标场景的对象

// reScene = CCTransitionFadeDown::create(t, s);

   CCDirector::sharedDirector()-&gt;replaceScene(reScene);

   CCTransitionTurnOffTiles

// 作用：创建一个随机方格消失的过渡动画

// 参数1：过渡动作的持续时间

// 参数2：切换的目标场景的对象

   reScene= CCTransitionTurnOffTiles::create(t, s);

   CCDirector::sharedDirector()-&gt;replaceScene(reScene);

// CCTransitionSplitRows

// 作用：创建一个分行划分切换的过渡动画

// 参数1：过渡动作的持续时间

// 参数2：切换的目标场景的对象

    reScene = CCTransitionSplitRows::create(t, s);

    CCDirector::sharedDirector()-&gt;replaceScene(reScene);

// CCTransitionSplitCols

// 作用：创建一个分列划分切换的过渡动画

// 参数1：过渡动作的持续时间

// 参数2：切换的目标场景的对象

    reScene = CCTransitionSplitCols::create(t, s);

    CCDirector::sharedDirector()-&gt;replaceScene(reScene);

// CCTransitionFade

// 作用：创建一个逐渐过渡到目标颜色的切换动画

// 参数1：过渡动作的持续时间

// 参数2：切换的目标场景的对象

// 参数3：目标颜色

    reScene= CCTransitionFade::create(t, s, ccc3(255, 0, 0));

    CCDirector::sharedDirector()-&gt;replaceScene(reScene);

// CCTransitionFlipX

// 作用：创建一个x轴反转的切换动画

// 参数1：过渡动作的持续时间

// 参数2：切换的目标场景的对象

// 参数3：反转类型的枚举变量 左右上下

// kOrientationDownOver kOrientationLeftOver kOrientationRightOver kOrientationUpOver

    reScene = CCTransitionFlipX::create(t, s, kOrientationRightOver);

    CCDirector::sharedDirector()-&gt;replaceScene(reScene);

// CCTransitionFlipY

// 参数1：过渡动作的持续时间

// 参数2：切换的目标场景的对象

// 参数3：反转类型的枚举变量 左右上下

    reScene = CCTransitionFlipY::create(t, s

   , kOrientationDownOver);

    CCDirector::sharedDirector()-&gt;replaceScene(reScene);

// CCTransitionFlipAngular

// 作用：创建一个带有反转角切换动画

// // 参数1：过渡动作的持续时间

// 参数2：切换的目标场景的对象

// 参数3：反转类型的枚举变量 左右上下

    reScene = CCTransitionFlipAngular::create(t, s, kOrientationLeftOver);

    CCDirector::sharedDirector()-&gt;replaceScene(reScene);

// CCTransitionZoomFlipX

// 作用：创建一个带有缩放的x轴反转切换的动画

// 参数1：过渡动作的持续时间

// 参数2：切换的目标场景的对象

// 参数3：反转类型的枚举变量 左右上下

    reScene=CCTransitionZoomFlipX::create(t, s, kOrientationLeftOver);

    CCDirector::sharedDirector()-&gt;replaceScene(reScene);

    CCTransitionZoomFlipY

// 作用：创建一个带有缩放的Y轴反转切换的动画

// 参数1：过渡动作的持续时间

// 参数2：切换的目标场景的对象

// 参数3：反转类型的枚举变量 左右上下

    reScene=CCTransitionZoomFlipY::create(t, s, kOrientationDownOver);

    CCDirector::sharedDirector()-&gt;replaceScene(reScene);

    CCTransitionZoomFlipAngular

// 作用：创建一个带有缩放 ，反转角切换的动画

// 参数1：过渡动作的持续时间

// 参数2：切换的目标场景的对象

// 参数3：反转类型的枚举变量 左右上下

   reScene=CCTransitionZoomFlipAngular::create(t, s, kOrientationRightOver);

   CCDirector::sharedDirector()-&gt;replaceScene(reScene);

// CCTransitionShrinkGrow

// 创建一个放缩交替的过渡动画

// 参数1：过渡动作的持续时间

// 参数2：切换的目标场景的对象

   reScene = CCTransitionShrinkGrow::create(t, s);

   CCDirector::sharedDirector()-&gt;replaceScene(reScene);

// CCTransitionRotoZoom

// 创建一个旋转放缩交替的过渡动画

// 参数1：过渡动作的持续时间

// 参数2：切换的目标场景的对象

    reScene = CCTransitionRotoZoom::create(t, s);

    CCDirector::sharedDirector()-&gt;replaceScene(reScene);

// CCTransitionMoveInL

// 作用：创建一个从左边推入覆盖的过渡动画

// 参数1：过渡动作的持续时间

// 参数2：切换的目标场景的对象

    reScene = CCTransitionMoveInL::create(t, s);

    CCDirector::sharedDirector()-&gt;replaceScene(reScene);

// CCTransitionMoveInR

// 作用：创建一个从右边推入覆盖的过渡动画

// 参数1：过渡动作的持续时间

// 参数2：切换的目标场景的对象

    reScene = CCTransitionMoveInR::create(t, s);

    CCDirector::sharedDirector()-&gt;replaceScene(reScene);

// CCTransitionMoveInB

// 作用：创建一个从下边推入覆盖的过渡动画

// 参数1：过渡动作的持续时间

// 参数2：切换的目标场景的对象

   reScene = CCTransitionMoveInB::create(t, s);

   CCDirector::sharedDirector()-&gt;replaceScene(reScene);

// CCTransitionMoveInT

// 作用：创建一个从上边推入覆盖的过渡动画

// 参数1：过渡动作的持续时间

// 参数2：切换的目标场景的对象

   reScene = CCTransitionMoveInT::create(t, s);

   CCDirector::sharedDirector()-&gt;replaceScene(reScene);

// CCTransitionSlideInL

// 作用：创建一个从左侧推入并顶出旧场景的过渡动画

// 参数1：过渡动作的持续时间

// 参数2：切换的目标场景的对象

   reScene =CCTransitionSlideInL::create(t, s);

   CCDirector::sharedDirector()-&gt;replaceScene(reScene);

// CCTransitionSlideInR

// 作用：创建一个从右侧推入并顶出旧场景的过渡动画

// 参数1：过渡动作的持续时间

// 参数2：切换的目标场景的对象

    reScene =CCTransitionSlideInR::create(t, s);

    CCDirector::sharedDirector()-&gt;replaceScene(reScene);

    CCTransitionSlideInT

// 作用：创建一个从顶部推入并顶出旧场景的过渡动画

// 参数1：过渡动作的持续时间

// 参数2：切换的目标场景的对象

   reScene =CCTransitionSlideInT::create(t, s);

   CCDirector::sharedDirector()-&gt;replaceScene(reScene);

   CCTransitionSlideInB

// 作用：创建一个从下部推入并顶出旧场景的过渡动画

// 参数1：过渡动作的持续时间

// 参数2：切换的目标场景的对象

   reScene =CCTransitionSlideInB::create(t, s);

   CCDirector::sharedDirector()-&gt;replaceScene(reScene);

}
```
---