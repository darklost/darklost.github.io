---
layout: post  
title: Cocos2d-x win32添加log窗口 自定义窗口尺寸参数
category: 游戏开发  
tags:  Cocos2d-x  	
keywords: cocos2dx win32  log winsize 
description:   
---

> cocos2d-x win32 添加log 窗口  添加自定义尺寸参数

`main.cpp`

```
#include "main.h"
#include "AppDelegate.h"
#include "CCEGLView.h"


USING_NS_CC;

#define USE_WIN32_CONSOLE  

#define DEFUALT_WIDTH  960;  
#define DEFUALT_HEIGHT  640;  

#define USE_WIN32_CONSOLE  
//字符串分割函数  
std::vector<std::string> split(std::string str, std::string pattern)
{
	std::string::size_type pos;
	std::vector<std::string> result;
	str += pattern;//扩展字符串以方便操作  
	int size = str.size();

	for (int i = 0; i<size; i++)
	{
		pos = str.find(pattern, i);
		if (pos<size)
		{
			std::string s = str.substr(i, pos - i);
			result.push_back(s);
			i = pos + pattern.size() - 1;
		}
	}
	return result;
}

int APIENTRY _tWinMain(HINSTANCE hInstance,
	HINSTANCE hPrevInstance,
	LPTSTR    lpCmdLine,
	int       nCmdShow)
{


	
	

	
	UNREFERENCED_PARAMETER(hPrevInstance);
	UNREFERENCED_PARAMETER(lpCmdLine);
	

#ifdef USE_WIN32_CONSOLE  
	AllocConsole();
	freopen("CONIN$", "r", stdin);
	freopen("CONOUT$", "w", stdout);
	freopen("CONOUT$", "w", stderr);
#endif  
	;
	// create the application instance
	int len = WideCharToMultiByte(CP_ACP, 0, lpCmdLine, -1, NULL, 0, NULL, NULL);
	char *ch = new char[len + 1];
	WideCharToMultiByte(CP_ACP, 0, lpCmdLine, -1, ch, len, NULL, NULL);
	std::string cmd(ch);
	delete[] ch;


	CCLOG("CMD =%s", cmd.c_str());
	std::vector <std::string> cmds= split(cmd," ");
	for (int i = 0; i < cmds.size(); i++)
	{
		CCLOG("cmds[%d] =%s", i, cmds[i].c_str());
	}
	int width = DEFUALT_WIDTH;
	int height = DEFUALT_HEIGHT;
	if (cmds.size()>0)
	{
		std::string size = cmds[0];
		std::vector <std::string> sizes = split(size, "x");
		if (sizes.size()==2)
		{
			width = atoi(sizes[0].c_str());
			height = atoi(sizes[1].c_str());
		}
	}


	
	AppDelegate app;
	CCEGLView* eglView = CCEGLView::sharedOpenGLView();
	eglView->setViewName("MaalaPanxi");
	eglView->setFrameSize(width, height);
	return CCApplication::sharedApplication()->run();

#ifdef USE_WIN32_CONSOLE  
	FreeConsole();
#endif  

}

```

---