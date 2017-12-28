---
layout: post
title: Git生成文件差异包
category: 技术
tags: Git
keywords: git patch diff
description: 
---

#### 1.需求

>生成git中两个版本有变化的文件,并将其打包zip
    
    
    

#### 2.实现1

使用

```

git archive -o ../lastest.zip HEAD $(git diff --name-only c5864b8f8b27a2b495f6bd051253166ec96b26ec)  


```

*结果:*

```
    fatal: pathspec 'setting/apk/README.MD' did not match any files
```

如果diff 出来的文件列表中有被删除的文件 则会打包失败

*也许 `git archive ` 有参数可以忽略不存在的目录 ；）*

    
    
    


#### 3.实现2

```

git diff c5864b8f8b27a2b495f6bd051253166ec96b26ec --name-only |xargs zip ../lastest.zip 

```


*结果:*

```
	zip warning: name not matched: setting/apk/README.MD
	zip warning: name not matched: setting/apk/app-release.apk
	zip warning: name not matched: setting/driver.txt
   adding: _posts/读书/2016-12-19-Book-List-2016.md (deflated 57%)
```

打包成功 不存在的文件会报警告 然后忽略

    
    
    


#### 4.more

git 文件中文路径

```
git config --global core.quotepath false

```


---
