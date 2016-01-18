---
layout: post  
title: C++通过技巧实现基本反射
category: 游戏开发  
tags: C++ 
keywords: C++ Reflector 
description:   
---

###1.基础类
```C++

#ifndef __Reflect_H__
#define __Reflect_H__

#include <string.h>

//Reflect base class
class reflect_object
{
  ;
};

//Reflect object creater
typedef reflect_object* (*object_newer) ();

//register of class
class class_register
{
public:
  class_register(class_register** p_context,
    const char* name,
    object_newer newer)
    :name(name), newer(newer)
  {
    //加入 reflection static linklist
    if (p_context != NULL)
    {
      //设置下一个当前类是传入进来的静态变量*p_context
      this->next = *p_context;
      *p_context = this;//*p_context指向当前实例
    }
  }
  reflect_object* new_object(const char* name)
  {
    //回调一下
    if (strcmp(this->name, name) == 0)
    {
      return this->newer();
    }

    //Call others (tail call)
    if (this->next == NULL)
    {
      return NULL;
    }
    return this->next->new_object(name);
  }
private:
  const char* name;
  object_newer newer;
  class_register* next;
};

//声明 reflect class
#define REFLECT_CLASS(class_name) class class_name:public reflect_object

//context of reflect ，全局共用context class_register (UI_ctx)
#define REFLECT_CONTEXT(context_name) static class_register* context_name = NULL;

//注册 reflect
#define REFLECT_REGIST(context_name,class_name) \
static reflect_object* context_name##_##class_name##_newer()    \
{                       \
  return new class_name();          \
}                       \   //静态初始化变量，此处传&(context_name) 是传引用保证初始化class_register 的时候用的是同一变量
static class_register cr_##class_name(&(context_name),#class_name,context_name##_##class_name##_newer);


//reflect 创建 
#define new_reflect_object(reflect_ctx,class_name) ((reflect_ctx)!=NULL?(reflect_ctx)->new_object(class_name):NULL)

REFLECT_CONTEXT(UI_ctx)

#endif //__Reflect_H__

```

###2.注册

```C++
#ifndef __POPRANK_UI_H__
#define __POPRANK_UI_H__

#include <stdio.h>
#include "Reflector.h"
#include "RankUI.h"

REFLECT_CLASS(PopRankUI)
{
public:
  void PopUI()
  {
    auto dir = Director::getInstance();
    auto curScene = dir->getRunningScene();

    RankUI* curui = RankUI::create();
    curScene->addChild(curui, 999);


    //出现的效果
    curui->m_pRootNode->getChildByName("Node")->setScale(0);
    curui->m_pRootNode->getChildByName("Node")->runAction(EaseElasticOut::create(ScaleTo::create(0.5, 1), 0.9));

    delete this;
  }
};



REFLECT_REGIST(UI_ctx, PopRankUI)

#endif

```

###3.调用

```C++

#include "PopShopUI.h"

//省略...

PopShopUI* instance=(PopShopUI*)new_reflect_object(UI_ctx, "PopShopUI");
instance->PopUI();

//省略...


```


---