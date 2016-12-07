---
layout: post  
title: Cocos2d-x emoj表情 部分Android机 crash
category: 游戏开发  
tags:  Cocos2d-x  	android
keywords: cocos2dx android  emoj crash 
description:   
---

> 解决 Cocos2d-x emoj表情 部分Android机 crash 

* 1 过滤emoj表情

* 2 修改引擎代码(基于cocos2d-x 2.2.6) 将jni 中传输 `jstring`  的 改用 `jbyteArray` 避免某些android 机型上 `NewStringUTF` 方法报错



`cocos2dx\platform\android\CCImage.cpp`

```
/****************************************************************************
Copyright (c) 2010 cocos2d-x.org

http://www.cocos2d-x.org

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.
****************************************************************************/

//#define COCOS2D_DEBUG 1

#define __CC_PLATFORM_IMAGE_CPP__
#include "platform/CCImageCommon_cpp.h"
#include "platform/CCPlatformMacros.h"
#include "platform/CCImage.h"
#include "platform/CCFileUtils.h"
#include "jni/JniHelper.h"

#include <android/log.h>
#include <string.h>
#include <jni.h>

// prototype
void swapAlphaChannel(unsigned int *pImageMemory, unsigned int numPixels);

NS_CC_BEGIN

class BitmapDC
{
public:

    BitmapDC()
    : m_pData(NULL)
    , m_nWidth(0)
    , m_nHeight(0)
    {
    }

    ~BitmapDC(void)
    {
        if (m_pData)
        {
            delete [] m_pData;
        }
    }

    bool getBitmapFromJavaShadowStroke(	const char *text,
    									int nWidth,
    									int nHeight,
    									CCImage::ETextAlign eAlignMask,
    									const char * pFontName,
    									float fontSize,
    									float textTintR 		= 1.0,
    									float textTintG 		= 1.0,
    									float textTintB 		= 1.0,
    									bool shadow 			= false,
    									float shadowDeltaX 		= 0.0,
    									float shadowDeltaY 		= 0.0,
    									float shadowBlur 		= 0.0,
    									float shadowIntensity 	= 0.0,
    									bool stroke 			= false,
    									float strokeColorR 		= 0.0,
    									float strokeColorG 		= 0.0,
    									float strokeColorB 		= 0.0,
    									float strokeSize 		= 0.0 )
    {
           JniMethodInfo methodInfo;
           if (! JniHelper::getStaticMethodInfo(methodInfo, "org/cocos2dx/lib/Cocos2dxBitmap", "createTextBitmapShadowStroke",
			   "([BLjava/lang/String;IFFFIIIZFFFZFFFF)V"))
           {
               CCLOG("%s %d: error to get methodInfo", __FILE__, __LINE__);
               return false;
           }
        
        
           // Do a full lookup for the font path using CCFileUtils in case the given font name is a relative path to a font file asset,
           // or the path has been mapped to a different location in the app package:
           std::string fullPathOrFontName = CCFileUtils::sharedFileUtils()->fullPathForFilename(pFontName);
        
		   // If the path name returned includes the 'assets' dir then that needs to be removed, because the android.content.Context
		   // requires this portion of the path to be omitted for assets inside the app package.
		   if (fullPathOrFontName.find("assets/") == 0)
		   {
               fullPathOrFontName = fullPathOrFontName.substr(strlen("assets/"));	// Chop out the 'assets/' portion of the path.
           }
           /**create bitmap
            * this method call Cococs2dx.createBitmap()(java code) to create the bitmap, the java code
            * will call Java_org_cocos2dx_lib_Cocos2dxBitmap_nativeInitBitmapDC() to init the width, height
            * and data.
            * use this approach to decrease the jni call number
           */
		

           //jstring jstrText = methodInfo.env->NewStringUTF(text);
		   //fix Android L  utf8 emoj erro
		   int strLen = strlen(text);
		   jbyteArray byteArray = methodInfo.env->NewByteArray(strLen);
		   methodInfo.env->SetByteArrayRegion(byteArray, 0, strLen, reinterpret_cast<const jbyte*>(text));

           jstring jstrFont = methodInfo.env->NewStringUTF(fullPathOrFontName.c_str());
		
		   methodInfo.env->CallStaticVoidMethod(methodInfo.classID, methodInfo.methodID, byteArray,
               jstrFont, (int)fontSize, textTintR, textTintG, textTintB, eAlignMask, nWidth, nHeight, shadow, shadowDeltaX, -shadowDeltaY, shadowBlur, stroke, strokeColorR, strokeColorG, strokeColorB, strokeSize);

           //methodInfo.env->DeleteLocalRef(jstrText);
		   methodInfo.env->DeleteLocalRef(byteArray);

           methodInfo.env->DeleteLocalRef(jstrFont);
           methodInfo.env->DeleteLocalRef(methodInfo.classID);
           return true;
    }


    bool getBitmapFromJava(const char *text, int nWidth, int nHeight, CCImage::ETextAlign eAlignMask, const char * pFontName, float fontSize)
    {
    	return  getBitmapFromJavaShadowStroke(	text, nWidth, nHeight, eAlignMask, pFontName, fontSize );
    }

    // ARGB -> RGBA
    inline unsigned int swapAlpha(unsigned int value)
    {
        return ((value << 8 & 0xffffff00) | (value >> 24 & 0x000000ff));
    }

public:
    int m_nWidth;
    int m_nHeight;
    unsigned char *m_pData;
    JNIEnv *env;
};

static BitmapDC& sharedBitmapDC()
{
    static BitmapDC s_BmpDC;
    return s_BmpDC;
}

bool CCImage::initWithString(
                               const char *    pText, 
                               int             nWidth/* = 0*/, 
                               int             nHeight/* = 0*/,
                               ETextAlign      eAlignMask/* = kAlignCenter*/,
                               const char *    pFontName/* = nil*/,
                               int             nSize/* = 0*/)
{
    bool bRet = false;

    do 
    {
        CC_BREAK_IF(! pText);
        
        BitmapDC &dc = sharedBitmapDC();

        CC_BREAK_IF(! dc.getBitmapFromJava(pText, nWidth, nHeight, eAlignMask, pFontName, nSize));

        // assign the dc.m_pData to m_pData in order to save time
        m_pData = dc.m_pData;
        CC_BREAK_IF(! m_pData);

        m_nWidth    = (short)dc.m_nWidth;
        m_nHeight   = (short)dc.m_nHeight;
        m_bHasAlpha = true;
        m_bPreMulti = true;
        m_nBitsPerComponent = 8;

        bRet = true;
    } while (0);

    return bRet;
}

bool CCImage::initWithStringShadowStroke(
                                         const char * pText,
                                         int         nWidth ,
                                         int         nHeight ,
                                         ETextAlign eAlignMask ,
                                         const char * pFontName ,
                                         int          nSize ,
                                         float        textTintR,
                                         float        textTintG,
                                         float        textTintB,
                                         bool shadow,
                                         float shadowOffsetX,
                                         float shadowOffsetY,
                                         float shadowOpacity,
                                         float shadowBlur,
                                         bool  stroke,
                                         float strokeR,
                                         float strokeG,
                                         float strokeB,
                                         float strokeSize)
{
	 bool bRet = false;
	    do
	    {
	        CC_BREAK_IF(! pText);

	        BitmapDC &dc = sharedBitmapDC();


	        CC_BREAK_IF(! dc.getBitmapFromJavaShadowStroke(pText, nWidth, nHeight, eAlignMask, pFontName,
	        											   nSize, textTintR, textTintG, textTintB, shadow,
	        											   shadowOffsetX, shadowOffsetY, shadowBlur, shadowOpacity,
	        											   stroke, strokeR, strokeG, strokeB, strokeSize ));


	        // assign the dc.m_pData to m_pData in order to save time
	        m_pData = dc.m_pData;

	        CC_BREAK_IF(! m_pData);

	        m_nWidth    = (short)dc.m_nWidth;
	        m_nHeight   = (short)dc.m_nHeight;
	        m_bHasAlpha = true;
	        m_bPreMulti = true;
	        m_nBitsPerComponent = 8;

	        // swap the alpha channel (ARGB to RGBA)
	        swapAlphaChannel((unsigned int *)m_pData, (m_nWidth * m_nHeight) );

	        // ok
	        bRet = true;

	    } while (0);

	    return bRet;
}

NS_CC_END

// swap the alpha channel in an 32 bit image (from ARGB to RGBA)
void swapAlphaChannel(unsigned int *pImageMemory, unsigned int numPixels)
{
	for(int c = 0; c < numPixels; ++c, ++pImageMemory)
	{
		// copy the current pixel
		unsigned int currenPixel =  (*pImageMemory);
		// swap channels and store back
		unsigned char *pSource = (unsigned char *) 	&currenPixel;
		*pImageMemory = (pSource[0] << 24) | (pSource[3]<<16) | (pSource[2]<<8) | pSource[1];
	}
}

// this method is called by Cocos2dxBitmap
extern "C"
{
    /**
    * this method is called by java code to init width, height and pixels data
    */
    JNIEXPORT void JNICALL Java_org_cocos2dx_lib_Cocos2dxBitmap_nativeInitBitmapDC(JNIEnv*  env, jobject thiz, int width, int height, jbyteArray pixels)
    {
        int size = width * height * 4;
        cocos2d::BitmapDC& bitmapDC = cocos2d::sharedBitmapDC();
        bitmapDC.m_nWidth = width;
        bitmapDC.m_nHeight = height;
        bitmapDC.m_pData = new unsigned char[size];
        env->GetByteArrayRegion(pixels, 0, size, (jbyte*)bitmapDC.m_pData);

        // swap data
        unsigned int *tempPtr = (unsigned int*)bitmapDC.m_pData;
        unsigned int tempdata = 0;
        for (int i = 0; i < height; ++i)
        {
            for (int j = 0; j < width; ++j)
            {
                tempdata = *tempPtr;
                *tempPtr++ = bitmapDC.swapAlpha(tempdata);
            }
        }
    }
};


```



`cocos2dx\platform\android\jni\Java_org_cocos2dx_lib_Cocos2dxHelper.cpp`

```

#include <stdlib.h>
#include <jni.h>
#include <android/log.h>
#include <string>
#include "JniHelper.h"
#include "cocoa/CCString.h"
#include "Java_org_cocos2dx_lib_Cocos2dxHelper.h"


#define  LOG_TAG    "Java_org_cocos2dx_lib_Cocos2dxHelper.cpp"
#define  LOGD(...)  __android_log_print(ANDROID_LOG_DEBUG,LOG_TAG,__VA_ARGS__)

#define  CLASS_NAME "org/cocos2dx/lib/Cocos2dxHelper"

static EditTextCallback s_pfEditTextCallback = NULL;
static void* s_ctx = NULL;

using namespace cocos2d;
using namespace std;

string g_apkPath;

extern "C" {

    JNIEXPORT void JNICALL Java_org_cocos2dx_lib_Cocos2dxHelper_nativeSetApkPath(JNIEnv*  env, jobject thiz, jstring apkPath) {
        g_apkPath = JniHelper::jstring2string(apkPath);
    }

    JNIEXPORT void JNICALL Java_org_cocos2dx_lib_Cocos2dxHelper_nativeSetEditTextDialogResult(JNIEnv * env, jobject obj, jbyteArray text) {
        jsize  size = env->GetArrayLength(text);

        if (size > 0) {
            jbyte * data = (jbyte*)env->GetByteArrayElements(text, 0);
            char* pBuf = (char*)malloc(size+1);
            if (pBuf != NULL) {
                memcpy(pBuf, data, size);
                pBuf[size] = '\0';
                // pass data to edittext's delegate
                if (s_pfEditTextCallback) s_pfEditTextCallback(pBuf, s_ctx);
                free(pBuf);
            }
            env->ReleaseByteArrayElements(text, data, 0);
        } else {
            if (s_pfEditTextCallback) s_pfEditTextCallback("", s_ctx);
        }
    }

}

const char * getApkPath() {
    return g_apkPath.c_str();
}

void showDialogJNI(const char * pszMsg, const char * pszTitle) {
    if (!pszMsg) {
        return;
    }

    JniMethodInfo t;
    if (JniHelper::getStaticMethodInfo(t, CLASS_NAME, "showDialog", "([B[B)V")) {

      

		

		int strLen1 = strlen(pszTitle);
		jbyteArray byteArray1 = t.env->NewByteArray(strLen1);
		t.env->SetByteArrayRegion(byteArray1, 0, strLen1, reinterpret_cast<const jbyte*>(pszTitle));

     


		int strLen2 = strlen(pszMsg);
		jbyteArray byteArray2 = t.env->NewByteArray(strLen2);
		t.env->SetByteArrayRegion(byteArray2, 0, strLen2, reinterpret_cast<const jbyte*>(pszMsg));

		t.env->CallStaticVoidMethod(t.classID, t.methodID, byteArray1, byteArray2);

		t.env->DeleteLocalRef(byteArray1);
		t.env->DeleteLocalRef(byteArray2);
        t.env->DeleteLocalRef(t.classID);
    }
}

void showEditTextDialogJNI(const char* pszTitle, const char* pszMessage, int nInputMode, int nInputFlag, int nReturnType, int nMaxLength, EditTextCallback pfEditTextCallback, void* ctx) {
    if (pszMessage == NULL) {
        return;
    }

    s_pfEditTextCallback = pfEditTextCallback;
    s_ctx = ctx;

    JniMethodInfo t;
    if (JniHelper::getStaticMethodInfo(t, CLASS_NAME, "showEditTextDialog", "([B[BIIII)V")) {
        jstring stringArg1;

		int strLen1 = strlen(pszTitle);
		jbyteArray byteArray1 = t.env->NewByteArray(strLen1);
		t.env->SetByteArrayRegion(byteArray1, 0, strLen1, reinterpret_cast<const jbyte*>(pszTitle));




		int strLen2 = strlen(pszMessage);
		jbyteArray byteArray2 = t.env->NewByteArray(strLen2);
		t.env->SetByteArrayRegion(byteArray2, 0, strLen2, reinterpret_cast<const jbyte*>(pszMessage));

		t.env->CallStaticVoidMethod(t.classID, t.methodID, byteArray1, byteArray2, nInputMode, nInputFlag, nReturnType, nMaxLength);

		t.env->DeleteLocalRef(byteArray1);
		t.env->DeleteLocalRef(byteArray2);
        t.env->DeleteLocalRef(t.classID);
    }
}

void terminateProcessJNI() {
    JniMethodInfo t;

    if (JniHelper::getStaticMethodInfo(t, CLASS_NAME, "terminateProcess", "()V")) {
        t.env->CallStaticVoidMethod(t.classID, t.methodID);
        t.env->DeleteLocalRef(t.classID);
    }
}

std::string getPackageNameJNI() {
    JniMethodInfo t;
    std::string ret("");

    if (JniHelper::getStaticMethodInfo(t, CLASS_NAME, "getCocos2dxPackageName", "()Ljava/lang/String;")) {
        jstring str = (jstring)t.env->CallStaticObjectMethod(t.classID, t.methodID);
        t.env->DeleteLocalRef(t.classID);
        ret = JniHelper::jstring2string(str);
        t.env->DeleteLocalRef(str);
    }
    return ret;
}

std::string getFileDirectoryJNI() {
    JniMethodInfo t;
    std::string ret("");

    if (JniHelper::getStaticMethodInfo(t, CLASS_NAME, "getCocos2dxWritablePath", "()Ljava/lang/String;")) {
        jstring str = (jstring)t.env->CallStaticObjectMethod(t.classID, t.methodID);
        t.env->DeleteLocalRef(t.classID);
        ret = JniHelper::jstring2string(str);
        t.env->DeleteLocalRef(str);
    }
    
    return ret;
}

std::string getCurrentLanguageJNI() {
    JniMethodInfo t;
    std::string ret("");
    
    if (JniHelper::getStaticMethodInfo(t, CLASS_NAME, "getCurrentLanguage", "()Ljava/lang/String;")) {
        jstring str = (jstring)t.env->CallStaticObjectMethod(t.classID, t.methodID);
        t.env->DeleteLocalRef(t.classID);
        ret = JniHelper::jstring2string(str);
        t.env->DeleteLocalRef(str);
    }

    return ret;
}

void enableAccelerometerJNI() {
    JniMethodInfo t;

    if (JniHelper::getStaticMethodInfo(t, CLASS_NAME, "enableAccelerometer", "()V")) {
        t.env->CallStaticVoidMethod(t.classID, t.methodID);
        t.env->DeleteLocalRef(t.classID);
    }
}

void setAccelerometerIntervalJNI(float interval) {
    JniMethodInfo t;

    if (JniHelper::getStaticMethodInfo(t, CLASS_NAME, "setAccelerometerInterval", "(F)V")) {
        t.env->CallStaticVoidMethod(t.classID, t.methodID, interval);
        t.env->DeleteLocalRef(t.classID);
    }
}

void disableAccelerometerJNI() {
    JniMethodInfo t;

    if (JniHelper::getStaticMethodInfo(t, CLASS_NAME, "disableAccelerometer", "()V")) {
        t.env->CallStaticVoidMethod(t.classID, t.methodID);
        t.env->DeleteLocalRef(t.classID);
    }
}

// functions for CCUserDefault
bool getBoolForKeyJNI(const char* pKey, bool defaultValue)
{
    JniMethodInfo t;
    
    if (JniHelper::getStaticMethodInfo(t, CLASS_NAME, "getBoolForKey", "(Ljava/lang/String;Z)Z")) {
        jstring stringArg = t.env->NewStringUTF(pKey);
        jboolean ret = t.env->CallStaticBooleanMethod(t.classID, t.methodID, stringArg, defaultValue);
        
        t.env->DeleteLocalRef(t.classID);
        t.env->DeleteLocalRef(stringArg);
        
        return ret;
    }
    
    return defaultValue;
}

int getIntegerForKeyJNI(const char* pKey, int defaultValue)
{
    JniMethodInfo t;
    
    if (JniHelper::getStaticMethodInfo(t, CLASS_NAME, "getIntegerForKey", "(Ljava/lang/String;I)I")) {
        jstring stringArg = t.env->NewStringUTF(pKey);
        jint ret = t.env->CallStaticIntMethod(t.classID, t.methodID, stringArg, defaultValue);
        
        t.env->DeleteLocalRef(t.classID);
        t.env->DeleteLocalRef(stringArg);
        
        return ret;
    }
    
    return defaultValue;
}

float getFloatForKeyJNI(const char* pKey, float defaultValue)
{
    JniMethodInfo t;
    
    if (JniHelper::getStaticMethodInfo(t, CLASS_NAME, "getFloatForKey", "(Ljava/lang/String;F)F")) {
        jstring stringArg = t.env->NewStringUTF(pKey);
        jfloat ret = t.env->CallStaticFloatMethod(t.classID, t.methodID, stringArg, defaultValue);
        
        t.env->DeleteLocalRef(t.classID);
        t.env->DeleteLocalRef(stringArg);
        
        return ret;
    }
    
    return defaultValue;
}

double getDoubleForKeyJNI(const char* pKey, double defaultValue)
{
    JniMethodInfo t;
    
    if (JniHelper::getStaticMethodInfo(t, CLASS_NAME, "getDoubleForKey", "(Ljava/lang/String;D)D")) {
        jstring stringArg = t.env->NewStringUTF(pKey);
        jdouble ret = t.env->CallStaticDoubleMethod(t.classID, t.methodID, stringArg);
        
        t.env->DeleteLocalRef(t.classID);
        t.env->DeleteLocalRef(stringArg);
        
        return ret;
    }
    
    return defaultValue;
}

std::string getStringForKeyJNI(const char* pKey, const char* defaultValue)
{
    JniMethodInfo t;
    std::string ret("");

    if (JniHelper::getStaticMethodInfo(t, CLASS_NAME, "getStringForKey", "(Ljava/lang/String;Ljava/lang/String;)Ljava/lang/String;")) {
        jstring stringArg1 = t.env->NewStringUTF(pKey);
        jstring stringArg2 = t.env->NewStringUTF(defaultValue);
        jstring str = (jstring)t.env->CallStaticObjectMethod(t.classID, t.methodID, stringArg1, stringArg2);
        ret = JniHelper::jstring2string(str);
        
        t.env->DeleteLocalRef(t.classID);
        t.env->DeleteLocalRef(stringArg1);
        t.env->DeleteLocalRef(stringArg2);
        t.env->DeleteLocalRef(str);
        
        return ret;
    }
    
    return defaultValue;
}

void setBoolForKeyJNI(const char* pKey, bool value)
{
    JniMethodInfo t;
    
    if (JniHelper::getStaticMethodInfo(t, CLASS_NAME, "setBoolForKey", "(Ljava/lang/String;Z)V")) {
        jstring stringArg = t.env->NewStringUTF(pKey);
        t.env->CallStaticVoidMethod(t.classID, t.methodID, stringArg, value);
        
        t.env->DeleteLocalRef(t.classID);
        t.env->DeleteLocalRef(stringArg);
    }
}

void setIntegerForKeyJNI(const char* pKey, int value)
{
    JniMethodInfo t;
    
    if (JniHelper::getStaticMethodInfo(t, CLASS_NAME, "setIntegerForKey", "(Ljava/lang/String;I)V")) {
        jstring stringArg = t.env->NewStringUTF(pKey);
        t.env->CallStaticVoidMethod(t.classID, t.methodID, stringArg, value);
        
        t.env->DeleteLocalRef(t.classID);
        t.env->DeleteLocalRef(stringArg);
    }
}

void setFloatForKeyJNI(const char* pKey, float value)
{
    JniMethodInfo t;
    
    if (JniHelper::getStaticMethodInfo(t, CLASS_NAME, "setFloatForKey", "(Ljava/lang/String;F)V")) {
        jstring stringArg = t.env->NewStringUTF(pKey);
        t.env->CallStaticVoidMethod(t.classID, t.methodID, stringArg, value);
        
        t.env->DeleteLocalRef(t.classID);
        t.env->DeleteLocalRef(stringArg);
    }
}

void setDoubleForKeyJNI(const char* pKey, double value)
{
    JniMethodInfo t;
    
    if (JniHelper::getStaticMethodInfo(t, CLASS_NAME, "setDoubleForKey", "(Ljava/lang/String;D)V")) {
        jstring stringArg = t.env->NewStringUTF(pKey);
        t.env->CallStaticVoidMethod(t.classID, t.methodID, stringArg, value);
        
        t.env->DeleteLocalRef(t.classID);
        t.env->DeleteLocalRef(stringArg);
    }
}

void setStringForKeyJNI(const char* pKey, const char* value)
{
    JniMethodInfo t;
    
    if (JniHelper::getStaticMethodInfo(t, CLASS_NAME, "setStringForKey", "(Ljava/lang/String;Ljava/lang/String;)V")) {
        jstring stringArg1 = t.env->NewStringUTF(pKey);
        jstring stringArg2 = t.env->NewStringUTF(value);
        t.env->CallStaticVoidMethod(t.classID, t.methodID, stringArg1, stringArg2);
        
        t.env->DeleteLocalRef(t.classID);
        t.env->DeleteLocalRef(stringArg1);
        t.env->DeleteLocalRef(stringArg2);
    }
}


```


`cocos2dx\platform\android\java\src\org\cocos2dx\lib\Cocos2dxBitmap.java`

```

/****************************************************************************
Copyright (c) 2010-2011 cocos2d-x.org

http://www.cocos2d-x.org

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.
 ****************************************************************************/
package org.cocos2dx.lib;

import java.nio.ByteBuffer;
import java.nio.ByteOrder;
import java.util.LinkedList;

import android.content.Context;
import android.graphics.Bitmap;
import android.graphics.Canvas;
import android.graphics.Color;
import android.graphics.Paint;
import android.graphics.Paint.Align;
import android.graphics.Paint.FontMetricsInt;
import android.graphics.Rect;
import android.graphics.Typeface;
import android.text.TextPaint;
import android.text.TextUtils;
import android.util.FloatMath;
import android.util.Log;

public class Cocos2dxBitmap {
	// ===========================================================
	// Constants
	// ===========================================================

	/* The values are the same as cocos2dx/platform/CCImage.h. */
	private static final int HORIZONTALALIGN_LEFT = 1;
	private static final int HORIZONTALALIGN_RIGHT = 2;
	private static final int HORIZONTALALIGN_CENTER = 3;
	private static final int VERTICALALIGN_TOP = 1;
	private static final int VERTICALALIGN_BOTTOM = 2;
	private static final int VERTICALALIGN_CENTER = 3;

	// ===========================================================
	// Fields
	// ===========================================================

	private static Context sContext;

	// ===========================================================
	// Constructors
	// ===========================================================

	// ===========================================================
	// Getter & Setter
	// ===========================================================

	public static void setContext(final Context pContext) {
		Cocos2dxBitmap.sContext = pContext;
	}

	// ===========================================================
	// Methods for/from SuperClass/Interfaces
	// ===========================================================

	// ===========================================================
	// Methods
	// ===========================================================

	private static native void nativeInitBitmapDC(final int pWidth,
			final int pHeight, final byte[] pPixels);

	/**
	 * @param pWidth
	 *            the width to draw, it can be 0
	 * @param pHeight
	 *            the height to draw, it can be 0
	 */
	public static void createTextBitmap(String pString, final String pFontName,
			final int pFontSize, final int pAlignment, final int pWidth,
			final int pHeight) {
		
		//
		createTextBitmapShadowStroke( pString.getBytes(), pFontName, pFontSize, 1.0f, 1.0f, 1.0f,   	// text font and color
									  pAlignment, pWidth, pHeight,							// alignment and size
									  false, 0.0f, 0.0f, 0.0f,								// no shadow
									  false, 1.0f, 1.0f, 1.0f, 1.0f);						// no stroke
									 
	}

	public static void createTextBitmapShadowStroke(byte[] bytes,  final String pFontName, final int pFontSize,
													final float fontTintR, final float fontTintG, final float fontTintB,
													final int pAlignment, final int pWidth, final int pHeight, final boolean shadow,
													final float shadowDX, final float shadowDY, final float shadowBlur, final boolean stroke,
													final float strokeR, final float strokeG, final float strokeB, final float strokeSize) {

        String pString;
        if (bytes == null || bytes.length == 0) {
          pString = "";
        } else {
          pString = new String(bytes);
        }
		
		final int horizontalAlignment = pAlignment & 0x0F;
		final int verticalAlignment   = (pAlignment >> 4) & 0x0F;

		pString = Cocos2dxBitmap.refactorString(pString);
		final Paint paint = Cocos2dxBitmap.newPaint(pFontName, pFontSize, horizontalAlignment);
		
		// set the paint color
		paint.setARGB(255, (int)(255.0 * fontTintR), (int)(255.0 * fontTintG), (int)(255.0 * fontTintB));

		final TextProperty textProperty = Cocos2dxBitmap.computeTextProperty(pString, pWidth, pHeight, paint);
		final int bitmapTotalHeight = (pHeight == 0 ? textProperty.mTotalHeight: pHeight);
		
		// padding needed when using shadows (not used otherwise)
		float bitmapPaddingX   = 0.0f;
		float bitmapPaddingY   = 0.0f;
		float renderTextDeltaX = 0.0f;
		float renderTextDeltaY = 0.0f;
		
		if ( shadow ) {

			int shadowColor = 0xff7d7d7d;
			paint.setShadowLayer(shadowBlur, shadowDX, shadowDY, shadowColor);
	
			bitmapPaddingX = Math.abs(shadowDX);
			bitmapPaddingY = Math.abs(shadowDY);
					
			if ( shadowDX < 0.0 )
			{
				renderTextDeltaX = bitmapPaddingX;
			}
			
			if ( shadowDY < 0.0 )
			{
				renderTextDeltaY = 	bitmapPaddingY;
			}
		}
		
		final Bitmap bitmap = Bitmap.createBitmap(textProperty.mMaxWidth + (int)bitmapPaddingX,
				bitmapTotalHeight + (int)bitmapPaddingY, Bitmap.Config.ARGB_8888);
		
		final Canvas canvas = new Canvas(bitmap);

		/* Draw string. */
		final FontMetricsInt fontMetricsInt = paint.getFontMetricsInt();
		
		int x = 0;
		int y = Cocos2dxBitmap.computeY(fontMetricsInt, pHeight, textProperty.mTotalHeight, verticalAlignment);
		
		final String[] lines = textProperty.mLines;
		
		for (final String line : lines) {
			
			x = Cocos2dxBitmap.computeX(line, textProperty.mMaxWidth, horizontalAlignment);
			canvas.drawText(line, x + renderTextDeltaX, y + renderTextDeltaY, paint);
			y += textProperty.mHeightPerLine;
			
		}
		 
		// draw again with stroke on if needed 
		if ( stroke ) {
			
			final Paint paintStroke = Cocos2dxBitmap.newPaint(pFontName, pFontSize, horizontalAlignment);
			paintStroke.setStyle(Paint.Style.STROKE);
			paintStroke.setStrokeWidth(strokeSize * 0.5f);
			paintStroke.setARGB(255, (int)strokeR * 255, (int)strokeG * 255, (int)strokeB * 255);
			
			x = 0;
			y = Cocos2dxBitmap.computeY(fontMetricsInt, pHeight, textProperty.mTotalHeight, verticalAlignment);
			final String[] lines2 = textProperty.mLines;
			
			for (final String line : lines2) {
				
				x = Cocos2dxBitmap.computeX(line, textProperty.mMaxWidth, horizontalAlignment);
				canvas.drawText(line, x + renderTextDeltaX, y + renderTextDeltaY, paintStroke);
				y += textProperty.mHeightPerLine;
				
			}
			
		}
		
		Cocos2dxBitmap.initNativeObject(bitmap);
	}

	private static Paint newPaint(final String pFontName, final int pFontSize,
			final int pHorizontalAlignment) {
		final Paint paint = new Paint();
		paint.setColor(Color.WHITE);
		paint.setTextSize(pFontSize); 
		paint.setAntiAlias(true);

		/* Set type face for paint, now it support .ttf file. */
		if (pFontName.endsWith(".ttf")) {
			try {
				final Typeface typeFace = Cocos2dxTypefaces.get(
						Cocos2dxBitmap.sContext, pFontName);
				paint.setTypeface(typeFace);
			} catch (final Exception e) {
				Log.e("Cocos2dxBitmap", "error to create ttf type face: "
						+ pFontName);

				/* The file may not find, use system font. */
				paint.setTypeface(Typeface.create(pFontName, Typeface.NORMAL));
			}
		} else {
			paint.setTypeface(Typeface.create(pFontName, Typeface.NORMAL));
		}

		switch (pHorizontalAlignment) {
		case HORIZONTALALIGN_CENTER:
			paint.setTextAlign(Align.CENTER);
			break;
		case HORIZONTALALIGN_RIGHT:
			paint.setTextAlign(Align.RIGHT);
			break;
		case HORIZONTALALIGN_LEFT:
		default:
			paint.setTextAlign(Align.LEFT);
			break;
		}

		return paint;
	}
	
	private static TextProperty computeTextProperty(final String pString,
			final int pWidth, final int pHeight, final Paint pPaint) {
		final FontMetricsInt fm = pPaint.getFontMetricsInt();
		final int h = (int) Math.ceil(fm.bottom - fm.top);
		int maxContentWidth = 0;

		final String[] lines = Cocos2dxBitmap.splitString(pString, pWidth,
				pHeight, pPaint);

		if (pWidth != 0) {
			maxContentWidth = pWidth;
		} else {
			/* Compute the max width. */
			int temp = 0;
			for (final String line : lines) {
				temp = (int) FloatMath.ceil(pPaint.measureText(line, 0,
						line.length()));
				if (temp > maxContentWidth) {
					maxContentWidth = temp;
				}
			}
		}

		return new TextProperty(maxContentWidth, h, lines);
	}

	private static int computeX(final String pText, final int pMaxWidth,
			final int pHorizontalAlignment) {
		int ret = 0;

		switch (pHorizontalAlignment) {
		case HORIZONTALALIGN_CENTER:
			ret = pMaxWidth / 2;
			break;
		case HORIZONTALALIGN_RIGHT:
			ret = pMaxWidth;
			break;
		case HORIZONTALALIGN_LEFT:
		default:
			break;
		}

		return ret;
	}

	private static int computeY(final FontMetricsInt pFontMetricsInt,
			final int pConstrainHeight, final int pTotalHeight,
			final int pVerticalAlignment) {
		int y = -pFontMetricsInt.top;

		if (pConstrainHeight > pTotalHeight) {
			switch (pVerticalAlignment) {
			case VERTICALALIGN_TOP:
				y = -pFontMetricsInt.top;
				break;
			case VERTICALALIGN_CENTER:
				y = -pFontMetricsInt.top + (pConstrainHeight - pTotalHeight)
						/ 2;
				break;
			case VERTICALALIGN_BOTTOM:
				y = -pFontMetricsInt.top + (pConstrainHeight - pTotalHeight);
				break;
			default:
				break;
			}
		}

		return y;
	}

	/*
	 * If maxWidth or maxHeight is not 0, split the string to fix the maxWidth
	 * and maxHeight.
	 */
	private static String[] splitString(final String pString,
			final int pMaxWidth, final int pMaxHeight, final Paint pPaint) {
		final String[] lines = pString.split("\\n");
		String[] ret = null;
		final FontMetricsInt fm = pPaint.getFontMetricsInt();
		final int heightPerLine = (int) Math.ceil(fm.bottom - fm.top);
		final int maxLines = pMaxHeight / heightPerLine;

		if (pMaxWidth != 0) {
			final LinkedList<String> strList = new LinkedList<String>();
			for (final String line : lines) {
				/*
				 * The width of line is exceed maxWidth, should divide it into
				 * two or more lines.
				 */
				final int lineWidth = (int) FloatMath.ceil(pPaint
						.measureText(line));
				if (lineWidth > pMaxWidth) {
					strList.addAll(Cocos2dxBitmap.divideStringWithMaxWidth(
							line, pMaxWidth, pPaint));
				} else {
					strList.add(line);
				}

				/* Should not exceed the max height. */
				if (maxLines > 0 && strList.size() >= maxLines) {
					break;
				}
			}

			/* Remove exceeding lines. */
			if (maxLines > 0 && strList.size() > maxLines) {
				while (strList.size() > maxLines) {
					strList.removeLast();
				}
			}

			ret = new String[strList.size()];
			strList.toArray(ret);
		} else if (pMaxHeight != 0 && lines.length > maxLines) {
			/* Remove exceeding lines. */
			final LinkedList<String> strList = new LinkedList<String>();
			for (int i = 0; i < maxLines; i++) {
				strList.add(lines[i]);
			}
			ret = new String[strList.size()];
			strList.toArray(ret);
		} else {
			ret = lines;
		}

		return ret;
	}

	private static LinkedList<String> divideStringWithMaxWidth(
			final String pString, final int pMaxWidth, final Paint pPaint) {
		final int charLength = pString.length();
		int start = 0;
		int tempWidth = 0;
		final LinkedList<String> strList = new LinkedList<String>();

		/* Break a String into String[] by the width & should wrap the word. */
		for (int i = 1; i <= charLength; ++i) {
			tempWidth = (int) FloatMath.ceil(pPaint.measureText(pString, start,
					i));
			if (tempWidth >= pMaxWidth) {
				final int lastIndexOfSpace = pString.substring(0, i)
						.lastIndexOf(" ");

				if (lastIndexOfSpace != -1 && lastIndexOfSpace > start) {
					/* Should wrap the word. */
					strList.add(pString.substring(start, lastIndexOfSpace));
					i = lastIndexOfSpace + 1; // skip space
				} else {
					/* Should not exceed the width. */
					if (tempWidth > pMaxWidth) {
						strList.add(pString.substring(start, i - 1));
						/* Compute from previous char. */
						--i;
					} else {
						strList.add(pString.substring(start, i));
					}
				}

				/* Remove spaces at the beginning of a new line. */
				while (i < charLength && pString.charAt(i) == ' ') {
					++i;
				}

				start = i;
			}
		}

		/* Add the last chars. */
		if (start < charLength) {
			strList.add(pString.substring(start));
		}

		return strList;
	}

	private static String refactorString(final String pString) {
		/* Avoid error when content is "". */
		if (pString.compareTo("") == 0) {
			return " ";
		}

		/*
		 * If the font of "\n" is "" or "\n", insert " " in front of it. For
		 * example: "\nabc" -> " \nabc" "\nabc\n\n" -> " \nabc\n \n".
		 */
		final StringBuilder strBuilder = new StringBuilder(pString);
		int start = 0;
		int index = strBuilder.indexOf("\n");
		while (index != -1) {
			if (index == 0 || strBuilder.charAt(index - 1) == '\n') {
				strBuilder.insert(start, " ");
				start = index + 2;
			} else {
				start = index + 1;
			}

			if (start > strBuilder.length() || index == strBuilder.length()) {
				break;
			}

			index = strBuilder.indexOf("\n", start);
		}

		return strBuilder.toString();
	}

	private static void initNativeObject(final Bitmap pBitmap) {
		final byte[] pixels = Cocos2dxBitmap.getPixels(pBitmap);
		if (pixels == null) {
			return;
		}

		Cocos2dxBitmap.nativeInitBitmapDC(pBitmap.getWidth(),
				pBitmap.getHeight(), pixels);
	}

	private static byte[] getPixels(final Bitmap pBitmap) {
		if (pBitmap != null) {
			final byte[] pixels = new byte[pBitmap.getWidth()
					* pBitmap.getHeight() * 4];
			final ByteBuffer buf = ByteBuffer.wrap(pixels);
			buf.order(ByteOrder.nativeOrder());
			pBitmap.copyPixelsToBuffer(buf);
			return pixels;
		}

		return null;
	}

	private static int getFontSizeAccordingHeight(int height) {
		Paint paint = new Paint();
		Rect bounds = new Rect();

		paint.setTypeface(Typeface.DEFAULT);
		int incr_text_size = 1;
		boolean found_desired_size = false;

		while (!found_desired_size) {

			paint.setTextSize(incr_text_size);
			String text = "SghMNy";
			paint.getTextBounds(text, 0, text.length(), bounds);

			incr_text_size++;

			if (height - bounds.height() <= 2) {
				found_desired_size = true;
			}
			Log.d("font size", "incr size:" + incr_text_size);
		}
		return incr_text_size;
	}

	private static String getStringWithEllipsis(String pString, float width,
			float fontSize) {
		if (TextUtils.isEmpty(pString)) {
			return "";
		}

		TextPaint paint = new TextPaint();
		paint.setTypeface(Typeface.DEFAULT);
		paint.setTextSize(fontSize);

		return TextUtils.ellipsize(pString, paint, width,
				TextUtils.TruncateAt.END).toString();
	}

	// ===========================================================
	// Inner and Anonymous Classes
	// ===========================================================

	private static class TextProperty {
		/** The max width of lines. */
		private final int mMaxWidth;
		/** The height of all lines. */
		private final int mTotalHeight;
		private final int mHeightPerLine;
		private final String[] mLines;

		TextProperty(final int pMaxWidth, final int pHeightPerLine,
				final String[] pLines) {
			this.mMaxWidth = pMaxWidth;
			this.mHeightPerLine = pHeightPerLine;
			this.mTotalHeight = pHeightPerLine * pLines.length;
			this.mLines = pLines;
		}
	}
}


```


`cocos2dx\platform\android\java\src\org\cocos2dx\lib\Cocos2dxHelper.java`

```
/****************************************************************************
Copyright (c) 2010-2013 cocos2d-x.org

http://www.cocos2d-x.org

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.
 ****************************************************************************/
package org.cocos2dx.lib;

import java.io.UnsupportedEncodingException;
import java.util.Locale;

import android.app.Activity;
import android.content.Context;
import android.content.SharedPreferences;
import android.content.pm.ApplicationInfo;
import android.content.res.AssetManager;
import android.os.Build;
import android.util.DisplayMetrics;
import android.view.Display;
import android.view.WindowManager;

public class Cocos2dxHelper {
	// ===========================================================
	// Constants
	// ===========================================================
	private static final String PREFS_NAME = "Cocos2dxPrefsFile";

	// ===========================================================
	// Fields
	// ===========================================================

	private static Cocos2dxMusic sCocos2dMusic;
	private static Cocos2dxSound sCocos2dSound;
	private static AssetManager sAssetManager;
	private static Cocos2dxAccelerometer sCocos2dxAccelerometer;
	private static boolean sAccelerometerEnabled;
	private static String sPackageName;
	private static String sFileDirectory;
	private static Context sContext = null;
	private static Cocos2dxHelperListener sCocos2dxHelperListener;

	// ===========================================================
	// Constructors
	// ===========================================================

	public static void init(final Context pContext, final Cocos2dxHelperListener pCocos2dxHelperListener) {
		final ApplicationInfo applicationInfo = pContext.getApplicationInfo();
		
		Cocos2dxHelper.sContext = pContext;
		Cocos2dxHelper.sCocos2dxHelperListener = pCocos2dxHelperListener;

		Cocos2dxHelper.sPackageName = applicationInfo.packageName;
		Cocos2dxHelper.sFileDirectory = pContext.getFilesDir().getAbsolutePath();
		Cocos2dxHelper.nativeSetApkPath(applicationInfo.sourceDir);

		Cocos2dxHelper.sCocos2dxAccelerometer = new Cocos2dxAccelerometer(pContext);
		Cocos2dxHelper.sCocos2dMusic = new Cocos2dxMusic(pContext);
		int simultaneousStreams = Cocos2dxSound.MAX_SIMULTANEOUS_STREAMS_DEFAULT;
        if (Cocos2dxHelper.getDeviceModel().indexOf("GT-I9100") != -1) {
            simultaneousStreams = Cocos2dxSound.MAX_SIMULTANEOUS_STREAMS_I9100;
        }
        Cocos2dxHelper.sCocos2dSound = new Cocos2dxSound(pContext, simultaneousStreams);
		Cocos2dxHelper.sAssetManager = pContext.getAssets();
		Cocos2dxBitmap.setContext(pContext);
		Cocos2dxETCLoader.setContext(pContext);
	}

	// ===========================================================
	// Getter & Setter
	// ===========================================================

	// ===========================================================
	// Methods for/from SuperClass/Interfaces
	// ===========================================================

	// ===========================================================
	// Methods
	// ===========================================================

	private static native void nativeSetApkPath(final String pApkPath);

	private static native void nativeSetEditTextDialogResult(final byte[] pBytes);

	public static String getCocos2dxPackageName() {
		return Cocos2dxHelper.sPackageName;
	}

	public static String getCocos2dxWritablePath() {
		return Cocos2dxHelper.sFileDirectory;
	}

	public static String getCurrentLanguage() {
		return Locale.getDefault().getLanguage();
	}
	
	public static String getDeviceModel(){
		return Build.MODEL;
    }

	public static AssetManager getAssetManager() {
		return Cocos2dxHelper.sAssetManager;
	}

	public static void enableAccelerometer() {
		Cocos2dxHelper.sAccelerometerEnabled = true;
		Cocos2dxHelper.sCocos2dxAccelerometer.enable();
	}


	public static void setAccelerometerInterval(float interval) {
		Cocos2dxHelper.sCocos2dxAccelerometer.setInterval(interval);
	}

	public static void disableAccelerometer() {
		Cocos2dxHelper.sAccelerometerEnabled = false;
		Cocos2dxHelper.sCocos2dxAccelerometer.disable();
	}

	public static void preloadBackgroundMusic(final String pPath) {
		Cocos2dxHelper.sCocos2dMusic.preloadBackgroundMusic(pPath);
	}

	public static void playBackgroundMusic(final String pPath, final boolean isLoop) {
		Cocos2dxHelper.sCocos2dMusic.playBackgroundMusic(pPath, isLoop);
	}

	public static void resumeBackgroundMusic() {
		Cocos2dxHelper.sCocos2dMusic.resumeBackgroundMusic();
	}

	public static void pauseBackgroundMusic() {
		Cocos2dxHelper.sCocos2dMusic.pauseBackgroundMusic();
	}

	public static void stopBackgroundMusic() {
		Cocos2dxHelper.sCocos2dMusic.stopBackgroundMusic();
	}

	public static void rewindBackgroundMusic() {
		Cocos2dxHelper.sCocos2dMusic.rewindBackgroundMusic();
	}

	public static boolean isBackgroundMusicPlaying() {
		return Cocos2dxHelper.sCocos2dMusic.isBackgroundMusicPlaying();
	}

	public static float getBackgroundMusicVolume() {
		return Cocos2dxHelper.sCocos2dMusic.getBackgroundVolume();
	}

	public static void setBackgroundMusicVolume(final float volume) {
		Cocos2dxHelper.sCocos2dMusic.setBackgroundVolume(volume);
	}

	public static void preloadEffect(final String path) {
		Cocos2dxHelper.sCocos2dSound.preloadEffect(path);
	}

	public static int playEffect(final String path, final boolean isLoop) {
		return Cocos2dxHelper.sCocos2dSound.playEffect(path, isLoop);
	}

	public static void resumeEffect(final int soundId) {
		Cocos2dxHelper.sCocos2dSound.resumeEffect(soundId);
	}

	public static void pauseEffect(final int soundId) {
		Cocos2dxHelper.sCocos2dSound.pauseEffect(soundId);
	}

	public static void stopEffect(final int soundId) {
		Cocos2dxHelper.sCocos2dSound.stopEffect(soundId);
	}

	public static float getEffectsVolume() {
		return Cocos2dxHelper.sCocos2dSound.getEffectsVolume();
	}

	public static void setEffectsVolume(final float volume) {
		Cocos2dxHelper.sCocos2dSound.setEffectsVolume(volume);
	}

	public static void unloadEffect(final String path) {
		Cocos2dxHelper.sCocos2dSound.unloadEffect(path);
	}

	public static void pauseAllEffects() {
		Cocos2dxHelper.sCocos2dSound.pauseAllEffects();
	}

	public static void resumeAllEffects() {
		Cocos2dxHelper.sCocos2dSound.resumeAllEffects();
	}

	public static void stopAllEffects() {
		Cocos2dxHelper.sCocos2dSound.stopAllEffects();
	}

	public static void end() {
		Cocos2dxHelper.sCocos2dMusic.end();
		Cocos2dxHelper.sCocos2dSound.end();
	}

	public static void onResume() {
		if (Cocos2dxHelper.sAccelerometerEnabled) {
			Cocos2dxHelper.sCocos2dxAccelerometer.enable();
		}
	}

	public static void onPause() {
		if (Cocos2dxHelper.sAccelerometerEnabled) {
			Cocos2dxHelper.sCocos2dxAccelerometer.disable();
		}
	}

	public static void terminateProcess() {
		android.os.Process.killProcess(android.os.Process.myPid());
	}

	private static void showDialog(final byte[] pTitleBytes, final byte[] pMessageBytes) {
		
		String pTitle;
		if(pTitleBytes==null || pTitleBytes.length==0)
		{
			pTitle="";
		}else
		{
			pTitle=new String(pTitleBytes);
		}
		String pMessage;
		if(pMessageBytes==null || pMessageBytes.length==0)
		{
			pMessage="";
		}else
		{
			pMessage=new String(pMessageBytes);
		}
		
		Cocos2dxHelper.sCocos2dxHelperListener.showDialog(pTitle, pMessage);
	}

	private static void showEditTextDialog(final byte[] pTitleBytes, final byte[] pMessageBytes, final int pInputMode, final int pInputFlag, final int pReturnType, final int pMaxLength) {
		String pTitle;
		if(pTitleBytes==null || pTitleBytes.length==0)
		{
			pTitle="";
		}else
		{
			pTitle=new String(pTitleBytes);
		}
		
		String pMessage;
		if(pMessageBytes==null || pMessageBytes.length==0)
		{
			pMessage="";
		}else
		{
			pMessage=new String(pMessageBytes);
		}
		
		Cocos2dxHelper.sCocos2dxHelperListener.showEditTextDialog(pTitle, pMessage, pInputMode, pInputFlag, pReturnType, pMaxLength);
	}

	public static void setEditTextDialogResult(final String pResult) {
		try {
			final byte[] bytesUTF8 = pResult.getBytes("UTF8");

			Cocos2dxHelper.sCocos2dxHelperListener.runOnGLThread(new Runnable() {
				@Override
				public void run() {
					Cocos2dxHelper.nativeSetEditTextDialogResult(bytesUTF8);
				}
			});
		} catch (UnsupportedEncodingException pUnsupportedEncodingException) {
			/* Nothing. */
		}
	}

    public static int getDPI()
    {
		if (sContext != null)
		{
			DisplayMetrics metrics = new DisplayMetrics();
			WindowManager wm = ((Activity)sContext).getWindowManager();
			if (wm != null)
			{
				Display d = wm.getDefaultDisplay();
				if (d != null)
				{
					d.getMetrics(metrics);
					return (int)(metrics.density*160.0f);
				}
			}
		}
		return -1;
    }
    
    // ===========================================================
 	// Functions for CCUserDefault
 	// ===========================================================
    
    public static boolean getBoolForKey(String key, boolean defaultValue) {
    	SharedPreferences settings = ((Activity)sContext).getSharedPreferences(Cocos2dxHelper.PREFS_NAME, 0);
    	return settings.getBoolean(key, defaultValue);
    }
    
    public static int getIntegerForKey(String key, int defaultValue) {
    	SharedPreferences settings = ((Activity)sContext).getSharedPreferences(Cocos2dxHelper.PREFS_NAME, 0);
    	return settings.getInt(key, defaultValue);
    }
    
    public static float getFloatForKey(String key, float defaultValue) {
    	SharedPreferences settings = ((Activity)sContext).getSharedPreferences(Cocos2dxHelper.PREFS_NAME, 0);
    	return settings.getFloat(key, defaultValue);
    }
    
    public static double getDoubleForKey(String key, double defaultValue) {
    	// SharedPreferences doesn't support saving double value
    	SharedPreferences settings = ((Activity)sContext).getSharedPreferences(Cocos2dxHelper.PREFS_NAME, 0);
    	return settings.getFloat(key, (float)defaultValue);
    }
    
    public static String getStringForKey(String key, String defaultValue) {
    	SharedPreferences settings = ((Activity)sContext).getSharedPreferences(Cocos2dxHelper.PREFS_NAME, 0);
    	return settings.getString(key, defaultValue);
    }
    
    public static void setBoolForKey(String key, boolean value) {
    	SharedPreferences settings = ((Activity)sContext).getSharedPreferences(Cocos2dxHelper.PREFS_NAME, 0);
    	SharedPreferences.Editor editor = settings.edit();
    	editor.putBoolean(key, value);
    	editor.commit();
    }
    
    public static void setIntegerForKey(String key, int value) {
    	SharedPreferences settings = ((Activity)sContext).getSharedPreferences(Cocos2dxHelper.PREFS_NAME, 0);
    	SharedPreferences.Editor editor = settings.edit();
    	editor.putInt(key, value);
    	editor.commit();
    }
    
    public static void setFloatForKey(String key, float value) {
    	SharedPreferences settings = ((Activity)sContext).getSharedPreferences(Cocos2dxHelper.PREFS_NAME, 0);
    	SharedPreferences.Editor editor = settings.edit();
    	editor.putFloat(key, value);
    	editor.commit();
    }
    
    public static void setDoubleForKey(String key, double value) {
    	// SharedPreferences doesn't support recording double value
    	SharedPreferences settings = ((Activity)sContext).getSharedPreferences(Cocos2dxHelper.PREFS_NAME, 0);
    	SharedPreferences.Editor editor = settings.edit();
    	editor.putFloat(key, (float)value);
    	editor.commit();
    }
    
    public static void setStringForKey(String key, String value) {
    	SharedPreferences settings = ((Activity)sContext).getSharedPreferences(Cocos2dxHelper.PREFS_NAME, 0);
    	SharedPreferences.Editor editor = settings.edit();
    	editor.putString(key, value);
    	editor.commit();
    }
	
	// ===========================================================
	// Inner and Anonymous Classes
	// ===========================================================

	public static interface Cocos2dxHelperListener {
		public void showDialog(final String pTitle, final String pMessage);
		public void showEditTextDialog(final String pTitle, final String pMessage, final int pInputMode, final int pInputFlag, final int pReturnType, final int pMaxLength);

		public void runOnGLThread(final Runnable pRunnable);
	}
}



```






---