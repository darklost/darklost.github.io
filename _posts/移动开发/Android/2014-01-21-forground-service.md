---
layout: post  
title: 使用startForeground让android服务前台运行,防止被各类安全软件杀死    
category: 移动开发  
tags: Android 
keywords: Android Foreground Service
description:   
---
后台运行的服务,被各种手机卫士和内存清理工具一清理或者onLowMemory时就被强行kill掉，有可能是系统回收内存的一种机制，要想避免这种情况可以通过startForeground让服务前台运行，当stopservice的时候通过stopForeground去掉。

在service 的onStratCommand中执行

```java
Notification notification = new Notification(R.drawable.ic_launcher,

                "XXXX",

                System.currentTimeMillis());

		PendingIntent pintent = PendingIntent.getService(this, 0, intent, 0);

		notification.setLatestEventInfo(this, "XXXX运行中",

		        "点击打开XX", pintent);

		//让该service前台运行，避免手机休眠时系统自动杀掉该服务

		//如果 id 为 0 ，那么状态栏的 notification 将不会显示。

		startForeground(startId, notification);


	@Override

	public void onDestroy() {

		stopForeground(true);

		super.onDestroy();

	}
	
```

---