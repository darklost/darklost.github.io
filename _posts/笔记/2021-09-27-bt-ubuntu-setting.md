---
layout: post  
title: ubuntu服务器安装设置
category: 笔记  
tags: 笔记 linux  ubuntu	
keywords:   ubuntu 安装设置 limits.conf
description: ubuntu 安装设置 limits.conf

---
## 启用`root`

### 设置`root`用户密码

```

passwd root

```

### 启用`root`密码

```

passwd -u root

```

### 允许`root`用户登录

修改`/etc/ssh/sshd_config`
```

vim /etc/ssh/sshd_config
##修改
PermitRootLogin yes #默认为 PermitRootLogin prohibit-password

```

### 删除`ubuntu`用户

1. 登出`ubuntu`
2. 登录`root`
3. 删除`ubuntu` 执行 `userdel -r ubuntu` ps: `-r`代表将用户文件夹也删除


## 设置 ulimit


### 默认`ulimit -a`

```
ulimit -a

core file size          (blocks, -c) 0
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 7873
max locked memory       (kbytes, -l) 16384
max memory size         (kbytes, -m) unlimited
open files                      (-n) 1024
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) 8192
cpu time               (seconds, -t) unlimited
max user processes              (-u) 7873
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited

```

### 查询文档 `man limits.conf`

### `/etc/security/limits.conf` 添加

```

* soft     nproc          655350
* hard     nproc          655350
* soft     nofile         655350
* hard     nofile         655350

root soft     nproc          655350
root hard     nproc          655350
root soft     nofile         655350
root hard     nofile         655350


```

### `/etc/pam.d/common-session` 添加

```
session required pam_limits.so

```

### 重新登录查询`ulimit -a`

```
ulimit -a
core file size          (blocks, -c) 0
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 31649
max locked memory       (kbytes, -l) 65536
max memory size         (kbytes, -m) unlimited
open files                      (-n) 655350
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) 8192
cpu time               (seconds, -t) unlimited
max user processes              (-u) 655350
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited

```


---
