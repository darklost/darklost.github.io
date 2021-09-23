---
layout: post  
title: homelab-搭建日志-0x00-开端
category: homelab  
tags: 笔记 homelab  	
keywords: homelab 搭建
description:  搭建自己的服务器 

---

## 起因

有这个想法的原因:

1. 已有的设备快扛不住了 

       - 主力开发机(15款的macbook pro)已经扛不住了，半个月前经历了电池鼓胀。
       - 备用开发机(19款Dell Latitude 7490)一年前就电池鼓胀，拆了电池了。;)看来我跟电池挺不对付的
       - 家用台式机(16年i7-5800K 980Ti 16G[32G]-DDR4  256G[256+2T]-SSD 2T-HDD(`希捷`我已经坏了三块了) 120[240]-一体水冷 ) 不适合办公 ps:`[]`中的是更换或升级的

2. 云服务又贵又不好用
       主要原因在于我自己,云服务要好的网络环境,所以...


研究了近一个月，最终还是觉得买一台二手服务器，搭建一套给自己开发和玩的环境。
倒不是因为价格原因，二手捡垃圾也就小几千。
主要是安置的地方，和后续的费用;)
ps：不过我已经决定先放公司了，白嫖- -！

## 目的

1. 搭建一套测试工作环境服务包括但是限定于以下:
       - linux服务器-ubuntu系统，跑测试
       - web服务器-跑nginx等提供内网服务 
       - jenkins-提供持续集成构建
       - git服务器-跑gitea，我实在不想老是折腾github了，原因你懂的
       - 软路由- pfSense OPNsense OpenWrt(LEDE) iKuai
       - DNS-做内网域名映射
       - DDNS-对外网映射(测试了下公司有公网IP)
       - windows服务器-跑编译
       - macOS-跑编译
       - NTP
       - DHCP
       - ... 


 2. 研究技术
       - 网络部署
       - 虚拟化
       - 集群
       - 硬件

## 系列文章

       - 0x00 [homelab-搭建日志-0x00-开端](/2021/09/24/homelab-begin.html)



       





---
