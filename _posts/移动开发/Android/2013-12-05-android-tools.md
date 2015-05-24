---
layout: post  
title: Android开发工具
category: 移动开发  
tags: Android Tools	安全 
keywords: Android Tools	
description:   
---
##开发工具
Android 源码

官方网站：[http://source.android.com](http://source.android.com "http://source.android.com")

- Android SDK：

官方网站：[http://developer.android.com/sdk/index.html](http://developer.android.com/sdk/index.html "http://developer.android.com/sdk/index.html")

- Android NDK：

官方网站：[http://developer.android.com/tools/sdk/ndk/index.html](http://developer.android.com/tools/sdk/ndk/index.html "http://developer.android.com/tools/sdk/ndk/index.html")

- apktools：

官方网站：[http://code.google.com/p/android-apktool/](http://code.google.com/p/android-apktool/ "http://code.google.com/p/android-apktool/")

- smali：

官方网站：[https://code.google.com/p/smali/](https://code.google.com/p/smali/)

- dex2jar：

官方网站：[http://code.google.com/p/dex2jar](http://code.google.com/p/dex2jar "http://code.google.com/p/dex2jar")

- XMLPrinter2.jar：

官方网站：[http://code.google.com/p/android4me/](http://code.google.com/p/android4me/ "http://code.google.com/p/android4me/")

----------

##逆向工具

`工具转载自52破解论坛`
###下载地址：
百度网盘[http://pan.baidu.com/s/1qWA1jPi](http://pan.baidu.com/s/1qWA1jPi) 密码: gn9i		
52网盘[http://down.52pojie.cn/Tools/Android_Tools/AndCrack_Tool.rar](http://down.52pojie.cn/Tools/Android_Tools/AndCrack_Tool.rar)
###第一部分：反编译&编译
  

**ApkIDE**：比较旧的Android反编译的IDE，集成功能各种方便，但是不知道扩展，慢慢落后了..

**AndroidKiller**：LB传奇哥做的，膜拜他的编程功底，任何功能分分钟搞定。该集成工具扩展性很好，最重要的是作者还在汲取建议持续更新，动态调试拭目以待啊；AndroidKiller跟新地址	
**ApkToolkit**：可以安装Framework框架进行反编译；
**ApkTool**：这个我主要是用它的签名功能、dex/odex->smali、smali->dex、dex->jar等；	

**ApkTool原生**：最近用的少了，方便查看错误信息.	
**Jeb**：主要用它来增强反编译Java代码的准确性，强大一逼；

###第二部分：修改工具
  

**Reflector**：这个工具是修改Unity3D游戏中Dll的主要工具（没在包里..安装版，自行下载）
**IDA**：不多说，非常强大，遗憾的是我用的不多..版本是6.5绿色版的，自行更新啊
**Winhex**：二进制编辑工具，ida分析之后，一般用这个修改；
**DexFixer**：如果直接修改了dex的二进制码，就需要用这个修复一下；

###第三部分：辅助工具
  

**ApkEncryptTool**：apk伪加密/解密工具，很少用；	
**Arm汇编转换器**：arm代码转操作码的工具。很常用的arm代码转换比较准确，最好还是找周边；	
**ASCII2UNICODE**：ASCII和UNICODE互转工具；	
**BCompare**：文件比较工具。一般用它取经啊..	
**Burpsuite Pro**：抓包工具（感觉渗透工具比较好），我不常用，最常用的是Fidller我电脑上是安装版；	
**getsign**：获取签名信息工具。不准确，当信息头是“308”的时候才是正确的；	
**Jd-Gui**：是Jar代码查看工具；	
**PYG_密码综合工具**：各种通用密码的加密解密；
**SQLiteExpertPro**：SQL.db数据文件查看工具。当分析本地数据的储存时，需要用到；

###第四部分：资源提取
  

**Extract**：Unity3D资源文件的提取；也可以用于Unity3D制作的网页游戏资源的提取；教程地址
**Extractor**：通用资源类型的提取，各种很多种的资源格式，如JPG/BMP/WAV等等；
**IrfanView**：特殊图片格式的查看工具，比较喜欢他的文件夹的预览功能；

另：在AndCrack-Tool\Tools\010Editor目录下，打包了010Editor和Android分析常用的格式模块，
       包括32/64位，自行选择配置：
  


---
说明：
>1.说明我是工具党啊
2.里面的工具并不都是最新版、修改版、优化版，使用中有啥更新了什么的，自行选择哈；
特别指出几个处理的地方哈：
①AndroidKiller中集成了几个常用的插入smali代码：
  

需要配置的：
  

>1.里面的Apktool也有几个，都是网上的开源编译的，我都弄混了，反编译不了就替换着来吧：Apktool更新地址
  

>2.Reflector安装一个吧，然后把里面的路径设置一下；
>
>3.所有的东西要么都是网上共享的 要么是自己搜刮别人的，不过我感觉吧，这些应该都可以放出的，有什么疑问的可以联系我哦；
>
>4.其他没指出的自行摸索哈 我只是提供了一个框架：

 


---