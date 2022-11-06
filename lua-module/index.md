# lua module

```lua
package1 = {}

package1.const = &#34;测试常量&#34;


function package1.func1()
    io.write(&#34;this is public func\n&#34;)
end

return package1
```

req.lua

```lua
require &#34;package1&#34;
package1.func1()
print(package1)
```

```
lc@lc-virtual-machine:~/lua$ lua pack1.lua 
this is public func
table: 0x5575766224a0
```

注意事项：

- 测试文件是和封装好的模块在同一个目录，否则引用时需要设置路径。

  ```lua
  package.path = &#39;/home/lc/lua/1/package1.lua;&#39;;
  
  require &#34;package1&#34;
  
  package1.func1()
  
  print(package1)
  ```

- 模块名称和文件名称必须相同




