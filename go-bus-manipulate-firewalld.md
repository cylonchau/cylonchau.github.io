# 使用go操作dbus


github https://github.com/godbus/dbus

增加一个端口

```go
package main

import (
	"github.com/godbus/dbus/v5"
)

func main() {
	cli, err := dbus.SystemBus()

	if err != nil {
		panic(err)
	}

	obj := cli.Object("org.fedoraproject.FirewallD1", "/org/fedoraproject/FirewallD1")
	call := obj.Call("oorg.fedoraproject.FirewallD1.zone.addPort", 0, "public", "81", "tcp", "30000")

	if call.Err != nil {
		panic(call.Err)
	}
}
```

go-dbus 简单教程 https://blog.csdn.net/mathmonkey/article/details/38095289


