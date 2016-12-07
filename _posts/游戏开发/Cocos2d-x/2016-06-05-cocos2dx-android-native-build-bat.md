---
layout: post  
title: Cocos2d-x android编译so库拷贝资源bat脚本
category: 游戏开发  
tags:  Cocos2d-x  	android
keywords: cocos2dx android  copyRes ndk 
description:   
---

> 解决 需要编译Android包时手动拷贝资源的问题（基于cocos2d-x2.2.6  cocos2d-x3.x 已结采用python脚本解决此问题）


>> 参考Quick-Cocos2dx 中的build_native.bat 脚本


`修改`


* 1 针对项目的资源目录进行了修改

* 2 添加了ndk-build clean 的处理 便于Eclispe 使用



`build_native.bat`

```

@echo off

::判断传入参数是否为clean 调用清理
set CMD=%1%
if "%CMD%"=="clean" (
echo ndk-build clean
ndk-build clean
goto :eof
)


set DIR=%~dp0
set APP_ROOT=%DIR%..\
set APP_ANDROID_ROOT=%DIR%


echo - config:
echo   APP_ROOT            = %APP_ROOT%
echo   APP_ANDROID_ROOT    = %APP_ANDROID_ROOT%

rem if dont use DEBUG, comments out codes below
::set NDK_DEBUG=1

echo - cleanup
::echo - cleanup bin
::if exist "%APP_ANDROID_ROOT%bin" rmdir /s /q "%APP_ANDROID_ROOT%bin"
::mkdir "%APP_ANDROID_ROOT%bin"
echo - cleanup assets\iphone
if exist "%APP_ANDROID_ROOT%assets\iphone" rmdir /s /q "%APP_ANDROID_ROOT%assets\iphone"
mkdir "%APP_ANDROID_ROOT%assets\iphone"


echo - copy resources
xcopy /s /q /y "%APP_ROOT%Resources\*.*" "%APP_ANDROID_ROOT%assets\"

del /f /s /q "%APP_ANDROID_ROOT%assets\*.dat"

echo ndk-build begin

ndk-build



```



---