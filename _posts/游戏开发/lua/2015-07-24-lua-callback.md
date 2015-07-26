---
layout: post  
title: 让Cocos2d-x lua中回调支持对象方法
category: 游戏开发  
tags: Lua
keywords: Cocos2d-x lua
description:
---
在 `Cocos2d-Lua `中，存在很多异步或延迟的操作，例如后台加载图片、等待一定时间执行代码等。这些功能的函数通常要求传入一个 function 作为参数。

-- 在后台加载一个图像，加载完成后输出消息

```lua
display.addImageAsync("hello.png", function()
    print("load hello.png completed")
end)
```

但如果我们希望这种回调支持一个对象方法，就有点小困难了。因为 Lua 的对象方法在调用时需要使用 object:method() 形式，而回调是无法支持这种格式的。

好在 `Lua `强大的闭包功能不但好用而且对性能无影响，所以我们可以将代码改写为：

```
local MyClass = class("MyClass")

function MyClass:print()
    print("load hello.png completed")
end


local my = MyClass.new()

display.addImageAsync("hello.png", function()
    my:print()
end)
```

原理非常简单，就是在匿名函数里调用对象方法而已。

Quick 框架里已经提供了更简单的使用方法 handler() 函数：

```
display.addImageAsync("hello.png", hander(my, my.print))
```
---
