---
layout: post  
title: Googel可访问地址ping检测脚本
category: 技术  
tags: google ip	
keywords: google ip	
description:   
---

```python
#!/usr/bin/python
#-*- coding:utf8 -*-

'''
#Created on 2015-06-02
# Site:darklost.me
# pings运行一般运行一次就够了，自己选一个ip段
# google ip地址段[更多有效地址段自行寻找]
# 64.233.189.0 - 64.233.191.255
# 58.123.102.0 - 58.123.102.255
# 210.242.125.0 - 210.242.125.255
# 64.233.162.0 - 64.233.162.255
window下应该只需要把ping参数-c 改成 -n，没测
'''

import time,os
start_Time=int(time.time()) #记录开始时间

# 生成ips
def pings():
    ips=open('host.txt','w')
    for ip3 in range(2, 254):
        ip='210.242.125.'+str(ip3)+"\n"
        # print  ip
        ips.write(ip)
    ips.close()

#判断文件中的ip是否能ping通，并且将通与不通的ip分别写到两个文件中
#文件中的ip一行一个
def ping_Test():
    ips=open('host.txt','r')
    ip_True = open('ip_True.txt','w')
    ip_False = open('ip_False.txt','w')
    count_True,count_False=0,0
    for ip in ips.readlines():
        ip = ip.replace('\n','')  #替换掉换行符
        return1=os.system('ping -c 2 -W 1 %s'%ip) #每个ip ping2次，等待时间为1s
        if return1:
            print 'ping %s is fail'%ip
            ip_False.write(ip+"\n")  #把ping不通的写到ip_False.txt中
            count_False += 1
        else:
            print 'ping %s is ok'%ip
            ip_True.write(ip+"\n")  #把ping通的ip写到ip_True.txt中
            count_True += 1
    ip_True.close()
    ip_False.close()
    ips.close()
    end_Time = int(time.time())  #记录结束时间
    print "time(秒)：",end_Time - start_Time,"s"  #打印并计算用的时间
    print "ping通数：",count_True,"   ping不通的ip数：",count_False

# pings()
ping_Test()

```


---
