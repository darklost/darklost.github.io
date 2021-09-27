---
layout: post  
title: 宝塔面板安装关闭绑定
category: 笔记  
tags: 笔记 linux  宝塔面板	
keywords:   宝塔面板
description:  宝塔面板关闭绑定

---
## 本地安装都需要绑定宝塔账号

#### 可以使用一下脚本安装，就无须绑定

ps: 升级后记得替换安装脚本`install-ubuntu_6.0.sh`下载地址

```

wget -O install.sh http://download.bt.cn/install/install-ubuntu_6.0.sh && sed -i '/bind.pl/d' install.sh && sudo bash install.sh

```




---
