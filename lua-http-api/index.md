# lua nginx api


```lua
location = /reqq {
	default_type text/plain;
	content_by_lua_block {
		ngx.req.read_body()
		local data = ngx.req.get_body_data()
                local args, err = ngx.req.get_uri_args()
		if not args then
			ngx.say(&#39;post fail&#39;)
			return
		end
		
        for key,v in pairs(args) do
			ngx.say(key,&#34;::&#34;,v,&#34;--&#34;)
		end
		ngx.say(data)
	}
}
```

### ngx.exec 内部重定向

```lua
location = /bb {
    default_type text/plain;
    content_by_lua_block{
        ngx.exec(&#39;/reqq?m=DaoShengPay3&amp;c=Pay&amp;orderNo=PH201910111&#39;)
    }
}

location = /reqq {
    default_type text/plain;
    content_by_lua_block {
       ngx.req.read_body()
       local data = ngx.req.get_body_data()
       local args, err = ngx.req.get_uri_args()
       if not args then
               ngx.say(&#39;post fail&#39;)
               return
       end
       for key,v in pairs(args) do
               ngx.say(key,&#34;::&#34;,v,&#34;--&#34;)
       end
       ngx.say(data)
    }
}
```

![1570889980396](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/1570889980396.png)

### ngx.log

```lua
location = /testlog {
    default_type text/plain;
    content_by_lua_block{
        ngx.say(&#39;hello lua log&#39;)
        ngx.log(ngx.ERR, &#39;print hello lua log&#39;)
    }
}
```

```log
2019/10/12 22:25:31 [error] 468#0: *248 [lua] content_by_lua(nginx.conf:44):3: print hello lua log, client: 203.90.247.72, server: test111.hou2008.com, request: &#34;GET /testlog HTTP/1.1&#34;, host: &#34;test111.hou2008.com:8181&#34;, referrer: &#34;http://test111.hou2008.com/testlog&#34;
```

# Nginx API for Lua 

https://github.com/openresty/lua-nginx-module 

https://www.zifangsky.cn/1024.html
