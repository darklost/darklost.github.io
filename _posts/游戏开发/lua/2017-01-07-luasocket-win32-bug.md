---
layout: post  
title: luasocket 在window平台无法收到服务器关闭的消息
category: 游戏开发  
tags: Lua
keywords: Cocos2d-x lua
description:
---

#### 1.起因
    
    
    用Quick-Cocos2d-Community 和服务器联调 mac 上一切正常
    windows下在服务器关闭的时候发现一直没有收到socket close 的消息

    当时我手上主要是在折腾服务端，其实可以放下不管，因为在要出包的iOS和Android平台都没有问题，不影响线上

    但是不方便客户端哥们进行调试，毕竟大部分哥们还是在win下工作

- - - 

#### 2.初步处理

将quick的framework中的SocketTcp中以下输出日志注释打开

```
	function SocketTCP:_tick(dt)
	local __body, __status, __partial = self.tcp:receive("*a")	-- read the package body
	//打开日志
	print("body:", __body, "__status:", __status, "__partial:", __partial)
    if __status == STATUS_CLOSED or __status == STATUS_NOT_CONNECTED then
    	self:close()
    	if self.isConnected then
    		self:_onDisconnect()
    	else
    		self:_connectFailure()
    	end
   		return
	end
    if 	(__body and string.len(__body) == 0) or
		(__partial and string.len(__partial) == 0)
	then return end
	if __body and __partial then __body = __body .. __partial end
	self:dispatchEvent({name=SocketTCP.EVENT_DATA, data=(__partial or __body), partial=__partial, body=__body})
end

```

运行 连上服务器 关闭服务器

日志中一直输出`__status:timeout`

猜测跟设置的超时未0有关
在以下代码中：

```
function SocketTCP:connect(__host, __port, __retryConnectWhenFailure)
	if __host then self.host = __host end
	if __port then self.port = __port end
	if __retryConnectWhenFailure ~= nil then self.isRetryConnect = __retryConnectWhenFailure end
	assert(self.host or self.port, "Host and port are necessary!")
	--printInfo("%s.connect(%s, %d)", self.name, self.host, self.port)
	if isIpv6(self.host) then
		self.tcp = socket.tcp6()
	else
		self.tcp = socket.tcp()
	end
	self.tcp:settimeout(0)//此处

	if not self:_checkConnect() then
		-- remove the old tcp scheduler
		if self.connectTimeTickScheduler then scheduler.unscheduleGlobal(self.connectTimeTickScheduler) end
		self.connectTimeTickScheduler = scheduler.scheduleUpdateGlobal(handler(self, self._connectTimeTick))
	end
end


```
`
尝试将`self.tcp:settimeout(0) `改为`self.tcp:settimeout(0.001) `

测试windows平台可以收到服务器断开的消息

改成一下代码

```
function SocketTCP:connect(__host, __port, __retryConnectWhenFailure)
	if __host then self.host = __host end
	if __port then self.port = __port end
	if __retryConnectWhenFailure ~= nil then self.isRetryConnect = __retryConnectWhenFailure end
	assert(self.host or self.port, "Host and port are necessary!")
	--printInfo("%s.connect(%s, %d)", self.name, self.host, self.port)
	if isIpv6(self.host) then
		self.tcp = socket.tcp6()
	else
		self.tcp = socket.tcp()
	end

	//change by darklost.me 2017-01-06 17:47:21
	if device.platform == "windows" then
		self.tcp:settimeout(0.0001)
	else
		self.tcp:settimeout(0)
	end
	

	if not self:_checkConnect() then
		-- remove the old tcp scheduler
		if self.connectTimeTickScheduler then scheduler.unscheduleGlobal(self.connectTimeTickScheduler) end
		self.connectTimeTickScheduler = scheduler.scheduleUpdateGlobal(handler(self, self._connectTimeTick))
	end
end
```

- - -

#### 3.经一步研究

按道理说通过以上的方法已经绕过这个问题 就可以不管了 毕竟不需要出win32的货 orz

但是生命不朽折腾不止

设置self.tcp:settimeout(0.0001) 以后即使这个时间再小也是会阻塞的 不再是（non-blocking）无阻塞的了

虽然是客户端但还是会有影响和其他平台有差异 影响开发对网络通信问题的判断

通过搜索引擎检索和GitHub上查issue 还真发现遇到相同问题的哥们

[ issue ](https://github.com/diegonehab/luasocket/issues/174)

按照这哥们的说法 注释掉这行代码就可以了

```
local data, status, partial = self.tcp:receive("_a")
if __status == STATUS_CLOSED or _status == STATUS_NOT_CONNECTED then
--remote was closed.
end
I use code above to check remote socket is closed，but the status is allways timeout because the code at wsocket.c（socket_waitfd）：
if (timeout_iszero(tm)) return IO_TIMEOUT; / optimize timeout == 0 case */
because my socket is non-blocking，so it allways return timeout here。after delete this line，it's return closed。

```

讨论的最后也没有给出解决的方案

倒是给我指明了方向

我review了`wsocket.c`的代码

突然看到下面这段代码

```

/*-------------------------------------------------------------------------*\
* Receive with timeout
\*-------------------------------------------------------------------------*/
int socket_recv(p_socket ps, char *data, size_t count, size_t *got, 
        p_timeout tm) 
{
    int err, prev = IO_DONE;
    *got = 0;
    if (*ps == SOCKET_INVALID) return IO_CLOSED;
    for ( ;; ) {
        int taken = recv(*ps, data, (int) count, 0);
        if (taken > 0) {
            *got = taken;
            return IO_DONE;
        }
        if (taken == 0) return IO_CLOSED;
        err = WSAGetLastError();
        /* On UDP, a connreset simply means the previous send failed. 
         * So we try again. 
         * On TCP, it means our socket is now useless, so the error passes. 
         * (We will loop again, exiting because the same error will happen) */
        if (err != WSAEWOULDBLOCK) {
            if (err != WSAECONNRESET || prev == WSAECONNRESET) return err;
            prev = err;
        }
        if ((err = socket_waitfd(ps, WAITFD_R, tm)) != IO_DONE) return err;
    }
}

```

其中的这段注释
> /* On UDP, a connreset simply means the previous send failed. 
>  * So we try again. 
>  * On TCP, it means our socket is now useless, so the error passes. 
>  * (We will loop again, exiting because the same error will happen) */

大概意思是tcp 中我们会在循环一次知道遇到相同的错误才退出

但是参考代码逻辑
遇到WSAECONNRESET这个错误一直不会return也不会再循环一次任何一次`if (err != WSAECONNRESET || prev == WSAECONNRESET) return err;`的判断是prev会一直是IO_DONE
也就是说

```
	if (err != WSAEWOULDBLOCK) {
            if (err != WSAECONNRESET || prev == WSAECONNRESET) return err;
            prev = err;
            //此处少了continue
        }

```

以下方法中也存在相同的问题

```
int socket_recvfrom(p_socket ps, char *data, size_t count, size_t *got, 
        SA *addr, socklen_t *len, p_timeout tm) 

```


修改成以下

```
/*-------------------------------------------------------------------------*\
* Receive with timeout
\*-------------------------------------------------------------------------*/
int socket_recv(p_socket ps, char *data, size_t count, size_t *got, 
        p_timeout tm) 
{
    int err, prev = IO_DONE;
    *got = 0;
    if (*ps == SOCKET_INVALID) return IO_CLOSED;
    for ( ;; ) {
        int taken = recv(*ps, data, (int) count, 0);
        if (taken > 0) {
            *got = taken;
            return IO_DONE;
        }
        if (taken == 0) return IO_CLOSED;
        err = WSAGetLastError();
        /* On UDP, a connreset simply means the previous send failed. 
         * So we try again. 
         * On TCP, it means our socket is now useless, so the error passes. 
         * (We will loop again, exiting because the same error will happen) */
        if (err != WSAEWOULDBLOCK) {
            if (err != WSAECONNRESET || prev == WSAECONNRESET) return err;
            prev = err;

            //change by  2017-1-7 01:59:38 darklost.me
			//settimeout(0) is always return IO_TIMEOUT when remote server is closed!
			//maybe missing continue here?
            continue;
        }
        if ((err = socket_waitfd(ps, WAITFD_R, tm)) != IO_DONE) return err;
    }
}

/*-------------------------------------------------------------------------*\
* Recvfrom with timeout
\*-------------------------------------------------------------------------*/
int socket_recvfrom(p_socket ps, char *data, size_t count, size_t *got, 
        SA *addr, socklen_t *len, p_timeout tm) 
{
    int err, prev = IO_DONE;
    *got = 0;
    if (*ps == SOCKET_INVALID) return IO_CLOSED;
    for ( ;; ) {
        int taken = recvfrom(*ps, data, (int) count, 0, addr, len);
        if (taken > 0) {
            *got = taken;
            return IO_DONE;
        }
        if (taken == 0) return IO_CLOSED;
        err = WSAGetLastError();
        /* On UDP, a connreset simply means the previous send failed. 
         * So we try again. 
         * On TCP, it means our socket is now useless, so the error passes.
         * (We will loop again, exiting because the same error will happen) */
        if (err != WSAEWOULDBLOCK) {
            if (err != WSAECONNRESET || prev == WSAECONNRESET) return err;
            prev = err;

            //change by  2017-1-7 01:59:38 darklost.me
			//settimeout(0) is always return IO_TIMEOUT when remote server is closed!
			//maybe missing continue here?
            continue;
        }
        if ((err = socket_waitfd(ps, WAITFD_R, tm)) != IO_DONE) return err;
    }
}


```

重新编译Quick模拟器 测试可行
没有详细测试，不是是否会存在其他问题orz

在上面那个issue中添加一个 [comment](https://github.com/diegonehab/luasocket/issues/174#issuecomment-270978262)

同时新建了一个[issue](https://github.com/diegonehab/luasocket/issues/202)

- - - 

#### 4.暂告一段落

	○|￣|_！！！~

---
