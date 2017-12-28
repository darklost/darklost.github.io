---
layout: post  
title: 禁止未预期的全局变量
category: 游戏开发  
tags: Lua
keywords: Cocos2d-x lua
description:
---

Lua 有一个特性就是默认定义的变量都是全局的。为了避免这一点，我们需要在定义变量时使用 local 关键字。

但难免会出现遗忘的情况，这时候出现的一些 bug 是很难查找的。所以我们可以采取一点小技巧，改变创建全局变量的方式。

```lua

local __g = _G

-- export global variable
cc.exports = {}
setmetatable(cc.exports, {
    __newindex = function(_, name, value)
        rawset(__g, name, value)
    end,

    __index = function(_, name)
        return rawget(__g, name)
    end
})

-- disable create unexpected global variable
setmetatable(__g, {
    __newindex = function(_, name, value)
        local msg = "USE 'cc.exports.%s = value' INSTEAD OF SET GLOBAL VARIABLE"
        error(string.format(msg, name), 0)
    end
})

```

增加上面的代码后，我们要再定义全局变量就会的得到一个错误信息。

但有时候全局变量是必须的，例如一些全局函数。我们可以使用新的定义方式：

```lua

-- export global
cc.exports.MY_GLOBAL = "hello"

-- use global
print(MY_GLOBAL)
-- or
print(_G.MY_GLOBAL)
-- or
print(cc.exports.MY_GLOBAL)

-- delete global
cc.exports.MY_GLOBAL = nil

-- global function
local function test_function_()
end
cc.exports.test_function = test_function_

-- if you set global variable, get an error
INVALID_GLOBAL = "no"

```

**注意：**

如果是用 `cocos2d-x` 或者 `quick-cocos2d-x`，那么只需要在 `main.lua` 里最开始的地方加一行：

`CC_DISABLE_GLOBAL = true`

程序就会自动禁止创建全局变量。

---
