---

layout: post  
title: Cocos2d-x 通用Android.mk	文件
category: 游戏开发  
tags: Android Cocos2d-x 	
keywords: cocos2dx android mk 
description:   

---

通用的Cocos2d-x Android.mk文件


```

LOCAL_PATH := $(call my-dir)

include $(CLEAR_VARS)

#NDK_MOUDLE_PATH的路径
$(call import-add-path,$(LOCAL_PATH)/../../cocos2d)
$(call import-add-path,$(LOCAL_PATH)/../../cocos2d/external)
$(call import-add-path,$(LOCAL_PATH)/../../cocos2d/cocos)

LOCAL_MODULE := cocos2dcpp_shared

LOCAL_MODULE_FILENAME := libcocos2dcpp

# 配置自己的源文件目录和源文件后缀名
MY_FILES_PATH  :=  $(LOCAL_PATH) \
                   $(LOCAL_PATH)/../../Classes

#需要添加的源文件后缀
MY_FILES_SUFFIX := %.cpp %.cc

# 递归遍历目录下的所有的文件
rwildcard=$(wildcard $1$2) $(foreach d,$(wildcard $1*),$(call rwildcard,$d/,$2))

# 获取相应的源文件
MY_ALL_FILES := $(foreach src_path,$(MY_FILES_PATH), $(call rwildcard,$(src_path),*.*) ) 
MY_ALL_FILES := $(MY_ALL_FILES:$(MY_CPP_PATH)/./%=$(MY_CPP_PATH)%)
MY_SRC_LIST  := $(filter $(MY_FILES_SUFFIX),$(MY_ALL_FILES)) 
MY_SRC_LIST  := $(MY_SRC_LIST:$(LOCAL_PATH)/%=%)

# 去除字串的重复单词
define uniq =
  $(eval seen :=)
  $(foreach _,$1,$(if $(filter $_,${seen}),,$(eval seen += $_)))
  ${seen}
endef

# 递归遍历获取所有目录
MY_ALL_DIRS := $(dir $(foreach src_path,$(MY_FILES_PATH), $(call rwildcard,$(src_path),*/) ) )
MY_ALL_DIRS := $(call uniq,$(MY_ALL_DIRS))

# 赋值给NDK编译系统
LOCAL_SRC_FILES  := $(MY_SRC_LIST)
LOCAL_C_INCLUDES := $(MY_ALL_DIRS)

# _COCOS_HEADER_ANDROID_BEGIN
# _COCOS_HEADER_ANDROID_END

# _COCOS_LIB_ANDROID_BEGIN

LOCAL_STATIC_LIBRARIES := cocos2dx_static


# _COCOS_LIB_ANDROID_END

include $(BUILD_SHARED_LIBRARY)


# _COCOS_LIB_IMPORT_ANDROID_BEGIN


$(call import-module,.)


# _COCOS_LIB_IMPORT_ANDROID_END


```

---