# lua cjson使用


cjson下载

https://github.com/mpx/lua-cjson.git


下载解压后，编译需要根据自己的lua环境以及操作系统修改Makefile的一些配置，不然容易出错。
以下是Makefile中的一些配置。

```c
LUA_VERSION =       5.2
TARGET =            cjson.so
PREFIX =            /usr/local
CJSON_LDFLAGS =     -shared
LUA_INCLUDE_DIR =   $(PREFIX)/include
LUA_CMODULE_DIR =   $(PREFIX)/lib/lua/$(LUA_VERSION)
LUA_MODULE_DIR =    $(PREFIX)/share/lua/$(LUA_VERSION)
LUA_BIN_DIR =       $(PREFIX)/bin
```


https://blog.gezhiqiang.com/2017/08/24/lua-cjson/


```lua
local cjson = require("cjson")

local obj = {
    id = 1,
    name = "zhangsan",
    age = nil,
    is_male = false,
    hobby = {"zhangsan","lisi","wangwu"}
}

local str = cjson.encode(obj)

ngx.say(str)
```
