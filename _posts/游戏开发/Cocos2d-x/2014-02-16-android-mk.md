---
layout: post  
title: Android.mk文件解析	
category: 游戏开发  
tags: Android Cocos2d-x 	
keywords: cocos2dx android mk 
description:   
---

在Android平台写c/c++程序的时候需要用到Android.mk(Makefile)，一般用来编译c/c++源码、引用第三方头文件和库，生成程序所需的so文件。下面是一个cocos2d-x游戏的Android.mk(删除了一些重复的东西)，一般默认在jni目录下：

```
#1 mk文件当前路径
LOCAL_PATH := $(call my-dir)
 
#2 自定义了一个all_cpp_files_recursively函数，递归遍历返回给定目录下所有C++源文件。
all_cpp_files_recursively = \
		$(eval src_files = $(wildcard $1/*.cpp)) \
		$(eval src_files = $(src_files:$(LOCAL_PATH)/%=%))$(src_files) \
		$(eval item_all = $(wildcard $1/*)) \
		$(foreach item, $(item_all) $(),\
		$(eval item := $(item:%.cpp=%)) \
		$(call all_cpp_files_recursively, $(item))\
 	)
 
#3 自定义了一个all_c_files_recursively 函数，递归遍历返回给定目录下所有C源文件。

	all_c_files_recursively = \ 
		$(eval src_files = $(wildcard $1/*.c)) \
		$(eval src_files = $(src_files:$(LOCAL_PATH)/%=%))$(src_files) \ 
		$(eval item_all = $(wildcard $1/*)) \ 
 		$(foreach item, $(item_all) $(),\ 
  		$(eval item := $(item:%.c=%)) \ 
  		$(call all_c_files_recursively, $(item))\ 
	 )
  
#4 声明一个预编译库的模块：共享库
include $(CLEAR_VARS)
LOCAL_MODULE := mytt
LOCAL_SRC_FILES := prebuilt/armeabi/libmytt.so
LOCAL_LDLIBS:= -L$(SYSROOT)/usr/lib -llog
include $(PREBUILT_SHARED_LIBRARY)
 
#5 声明一个预编译库的模块：共享库
include $(CLEAR_VARS)
LOCAL_MODULE := myts
LOCAL_SRC_FILES := prebuilt/armeabi/libmyts.so
LOCAL_LDLIBS:= -L$(SYSROOT)/usr/lib -llog
include $(PREBUILT_SHARED_LIBRARY)
 
#6 声明一个预编译库的模块：静态库
include $(CLEAR_VARS)
LOCAL_MODULE := mycs
LOCAL_SRC_FILES := ../../Classes/libtgcpapi/android/libmycs.a
include $(PREBUILT_STATIC_LIBRARY)
 
#7 共享库模块west_shared
include $(CLEAR_VARS)
LOCAL_MODULE := west_shared
LOCAL_MODULE_FILENAME := libwest
 
#8 将要编译打包到模块west_shared中的c/c++源码文件
LOCAL_SRC_FILES := $(call all_cpp_files_recursively,$(LOCAL_PATH)/west)
LOCAL_SRC_FILES += $(call all_cpp_files_recursively,$(LOCAL_PATH)/../../Classes)
LOCAL_SRC_FILES += $(call all_c_files_recursively,$(LOCAL_PATH)/../../Classes)
 
#9 头文件的搜索路径
LOCAL_C_INCLUDES := $(LOCAL_PATH)/../../Classes \
     $(LOCAL_PATH)/../../Classes/子目录 \
     ......
     $(LOCAL_PATH)/west \
     $(LOCAL_PATH)/../../../其他第三方库
      
#10 模块west_shared链接时需要使用的静态库
LOCAL_STATIC_LIBRARIES := mycs
#  模块west_shared运行时依赖的共享库，源码中调用了其暴露的接口，所以链接时就需要，否则会出错。
LOCAL_SHARED_LIBRARIES := myts
 
#11 跟LOCAL_STATIC_LIBRARIES一样，只不过包含了静态库的所有的源代码。
LOCAL_WHOLE_STATIC_LIBRARIES := cocos2dx_static
LOCAL_WHOLE_STATIC_LIBRARIES += cocosdenshion_static
LOCAL_WHOLE_STATIC_LIBRARIES += cocos_lua_static
LOCAL_WHOLE_STATIC_LIBRARIES += cocos_extension_static
 
#12 存在于系统目录下本模块需要连接的库。
LOCAL_LDFLAGS+= -Xlinker --allow-multiple-definition
 
#13 表示编译成共享库
include $(BUILD_SHARED_LIBRARY)
 
#14 编译模块时要使用的附加的链接器选项
#LOCAL_LDLIBS:=
# 将一个新的路径xxxx加入NDK_MODULE_PATH变量
$(call import-add-path,$(LOCAL_PATH)/../../../)
$(call import-add-path,$(LOCAL_PATH)/../../../cocos2dx/platform/third_party/android/prebuilt)
 
# 导入外部模块提供的.mk文件
$(call import-module,cocos2dx)
$(call import-module,CocosDenshion/android)
$(call import-module,scripting/lua/proj.android)
$(call import-module,extensions)
下面来简单解释一下这几行代码。
```
###`#1`
**需要了解的知识点(LOCAL_PATH、:=和=、call函数、my-dir)。**

(1)定义了`LOCAL_PATH` 变量，下面的一些函数和变量依赖于它，记住：LOCAL_PATH 必须放到所有include $(CLEAR_VARS)之前定义。	

(2)`:=`是即时求值，如：
 x := foo
 y := $(x) bar
 x := xyz
在上例中，y的值将会是 foo bar。
=是等整个makefile展开后，再决定变量的值，就是如果x在下文被改了，y的值也会被更新，如：
x = foo
y = $(x) bar
x = xyz
在上例中，y的值将会是 xyz bar ，而不是 foo bar 。
(3)call用于调用其它函数, 参数以逗号分隔，函数原型：
$(call <expression>，<parm1>，<parm2>，<parm3>，…)
当make执行这个函数时，<expression>参数中的变量，如$(1)，$(2)，$(3)等，会被参数<parm1>，<parm2>，<parm3>依次取代。
Makefile函数调用形式：$(<function> <arguments>)或${<function> <arguments>}，<function>就是函数名，<arguments>为函数的参数，参数间以逗号“,”分隔，而函数名和参数之间以“空格”分隔。函数调用以“$”开头，以圆括号或花括号把函数名和参数括起。call函数的返回值就是<expression>的返回值。
(4)my-dir由编译系统提供，返回 Android.mk 当前所在的路径(包含Android.mk文件的目录)。
 

### `#2`
 ***自定义了一个all_cpp_files_recursively函数，递归遍历返回给定目录下所有C++源文件，google一下有很多类似代码。***
 
**需要了解的知识点(`eval`函数、`wildcard`函数，`foreach`函数)**

1.	`eval`： 在Makefile中构造一个可变的规则结构关系（依赖关系链），其中可以使用其它变量和函数。函数“eval”对它的参数进行展开，展开的结果作为Makefile的一部分，make可以对展开内容进行语法解析。“eval”函数执行时会对它的参数进行两次展开。第一次展开过程发是由函数本身完成的，第二次是函数展开后的结果被作为Makefile内容时由make解析时展开的。其实就是动态生成Makefile脚本，类似js中的eval(执行字符串形式的js代码)，eval函数没有返回值。

2.	`wildcard`：用来扩展通配符。在Makefile中使用通配符时存在一定的局限性，通配符自动在规则中进行，但当通配符用在变量或者函数定义中时，通配符将失效，make不会将其展开，而是将其作为简单的字符串处理，这种情况下要使通配符有效，则需要使用wildcard函数。

3.	`foreach`：做循环用的，函数原型是`$(foreach <var>,<list>，<text>)`
`foreach`的意思是，把参数`<list>`中的单词逐一取出放到参数`<var> `所指定的变量中，然后再执行`<text>`所包含的表达式。每一次`<text>`会返回一个字符串，所返回的每个字符串会以空格分隔，当整个循环结束时，把`<text>`所返回的每个字符串所组成的字符串（以空格分隔）作为foreach函数的返回值。举一个简单的例子：
```
names := a b c d
files := $(foreach n,$(names),$(n).o)
$(files)的值是“a.o b.o c.o d.o”
```
4. `$(src_files:$(LOCAL_PATH)/%=%)`进行了文本替换，`”%”`表示一个或多个任意字符，这里是去掉cpp/c文件路径中的`$(LOCAL_PATH)`部分，因为`LOCAL_SRC_FILES`文件的路径需要是`Android.mk`的相对路径。
 

### `#3`. 自定义了一个all_c_files_recursively函数，递归遍历返回给定目录下所有C源文件。
 

### `#4 #5 #6`

***声明预编译库***

首先需要读取`CLEAR_VARS` 变量，是编译系统提供的，指向一个`GNU Makefile`脚本，为你清除一些`LOCAL_XXX`变量`(e.g. LOCAL_MODULE, LOCAL_SRC_FILES, LOCAL_STATIC_LIBRARIES, etc…)`，`LOCAL_PATH`除外，避免相互影响。因为所有的编译控制文件是在同一个GNU Make执行环境解析的，因此所有变量都是全局的。

PS:

* Android.mk中可以定义多个编译模块，每个编译模块都是以`include $(CLEAR_VARS)`开始,以`include $(BUILD_XXX)`结束，每个预编译的库，都必须声明为一个独立的模块 。

* `PREBUILT_SHARED_LIBRARY`(预编译共享库) 和 `PREBUILT_STATIC_LIBRARY`(预编译静态库) ：
* `Android NDK r5 `开始支持预编译库，这样你就可以把预编译库给第三方NDK开发者使用，而不用暴露源码，也可以使用预编译库来加速项目构建。
* `PREBUILT_SHARED_LIBRARY`的`LOCAL_SRC_FILES`必须是预编译共享库文件(eg：t.so文件)，`PREBUILT_STATIC_LIBRARY`必须是预编译静态库文件(eg：t.a文件)。
 

### `#11` 
这里需要注意`LOCAL_WHOLE_STATIC_LIBRARIES`和`LOCAL_STATIC_LIBRARIES`的区别：
前者会包含静态库的所有的源代码，后者会允许链接器移除一些dead code（没有使用的变量或函数，比如一些给共享库使用的接口）。
 

### `#12` 
`-Xlinker`表示它后面的参数是给链接器使用的，`–allow-multiple-definition`表示允许发生多重定义并且使用第一个定义（跟链接顺序有关）。
 

### `#13` 
读取`BUILD_SHARED_LIBRARY`变量，把`west_shared`编译成共享库，是编译系统提供的，指向一个`GNU Makefile`脚本，它负责收集自从上次调用`include $(CLEAR_VARS)`之后的定义的`LOCAL_XXXX`变量的所有信息，决定怎么正确的编译出所需的共享库。
 

### `#14`
  ***`NDK_MODULE_PATH`***是一个很重要的变量.
  
  当`android.mk`中使用了`$(call import-module,XXX)`函数引入外部模块文件时会用到，用以指示该往哪里去找这个`Android.mk`文件。
  
 `$(call import-module, 相对路径)`，如`$(call import-module,cocos2dx)`，cocos2dx是相对路径，绝对路径是`NDK_MODULE_PATH/cocos2dx`，`NDK_MODULE_PATH`可以在执行`ndk_build`时指定。如：
 
`ndk-build V=0 NDK_LOG=0 NDK_DEBUG=1 NDK_MODULE_PATH=D:WorkSpace/west_game/`
 

下面是ndk编译相关命令：

1.	`ndk-build NDK_LOG=1`

	用于配置LOG级别，打印ndk编译时的详细输出信息。

2. `ndk-build NDK_PROJECT_PATH=.`
 
	指定NDK编译的代码路径为当前目录，如果不配置，则必须把工程代码放到Android工程的jni目录下

3. `ndk-build APP_BUILD_SCRIPT=./Android.mk`

	指定NDK编译使用的Android.mk文件 ，默认是$PROJECT/jni/下

4. `ndk-build NDK_APP_APPLICATION_MK=./Application.mk`
 
	指定NDK编译使用的application.mk文件 ，默认是$PROJECT/jni/下

5. `ndk-build clean`

	清除所有编译出来的临时文件和目标文件

6. `ndk-build -B`
 
	强制重新编译已经编译完成的代码

7. `ndk-build NDK_DEBUG=1`

	执行 debug 编译

8. `ndk-build NDK_DEBUG=0`

	执行 release 编译

9. `ndk-build NDK_OUT=./mydir`
 
	指定编译生成的文件的存放位置

10. `ndk-build -C /opt/mydir/`

	到指定目录编译native代码

 

可以在`Android.mk`中打印log或某个变量的值，如打印`LOCAL_PATH`变量：

`$(warning “the value of LOCAL_PATH is $(LOCAL_PATH)”)`


如果想要系统的学习Makefile和了解Android.mk，可参考：

[Android.mk file syntax specification](http://www.kandroid.org/ndk/docs/ANDROID-MK.html)

[跟我一起写Makefile](http://blog.csdn.net/haoel/article/details/2886)

---