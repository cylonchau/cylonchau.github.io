# 使用go操作dbus


github https://github.com/godbus/dbus

增加一个端口

```go
package main

import (
	&#34;github.com/godbus/dbus/v5&#34;
)

func main() {
	cli, err := dbus.SystemBus()

	if err != nil {
		panic(err)
	}

	obj := cli.Object(&#34;org.fedoraproject.FirewallD1&#34;, &#34;/org/fedoraproject/FirewallD1&#34;)
	call := obj.Call(&#34;oorg.fedoraproject.FirewallD1.zone.addPort&#34;, 0, &#34;public&#34;, &#34;81&#34;, &#34;tcp&#34;, &#34;30000&#34;)

	if call.Err != nil {
		panic(call.Err)
	}
}
```

go-dbus 简单教程 https://blog.csdn.net/mathmonkey/article/details/38095289


