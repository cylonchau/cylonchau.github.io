# firewalld dbus接口使用指南



firewalld，一个基于动态区的iptables/nftables守护程序，自2009年左右开始开发，CentOS7基于 `firewalld-0.6.3` ， 发布于2018年10月11日。主要的开发人员是托马斯·沃纳，他目前为红帽公司工作。这是因为为Federal 18 的默认防火墙机制， 随后在 Rhel7 和 Centos 7 中使用。

firewalld比旧的 iptable 机制有许多优势。值得注意的是，它解决了 iptable 要求每次更改时重新启动防火墙的问题，从而中断了任何状态连接。它还提供了丰富的 D-Bus 方法、信号和属性。

这里并不是从firewalld操作使用方式来介绍firewalld的改名，想反，是介绍firewalld D-Bus API来检索信息或更改设置。



firewalld被配置为系统 D-Bus 服务，注意看 systemd file中的" `Type=dbus` "参数。

```
$ cat /usr/lib/systemd/system/firewalld.service
[Unit]
Description=firewalld - dynamic firewall daemon
Before=network-pre.target
Wants=network-pre.target
After=dbus.service
After=polkit.service
Conflicts=iptables.service ip6tables.service ebtables.service ipset.service
Documentation=man:firewalld(1)

[Service]
EnvironmentFile=-/etc/sysconfig/firewalld
ExecStart=/usr/sbin/firewalld --nofork --nopid $FIREWALLD_ARGS
ExecReload=/bin/kill -HUP $MAINPID
# supress to log debug and error output also to /var/log/messages
StandardOutput=null
StandardError=null
Type=dbus
BusName=org.fedoraproject.FirewallD1
KillMode=mixed

[Install]
WantedBy=multi-user.target
Alias=dbus-org.fedoraproject.FirewallD1.service
```

实际上，手动运行 `/usr/bin/python2 -Es /usr/sbin/firewalld --nofork --nopid --debug` 效果是一样的，这里的注册是通过dbus 高级API操作的。

此时由于已经了解到了，firewalld 服务 是基于D-Bus接口的，所以需要找到对应的 dbus interface

```
dbus-send --system --dest=org.freedesktop.DBus \
	--type=method_call --print-reply \
	/org/freedesktop/DBus org.freedesktop.DBus.ListNames | grep FirewallD
```

`org.fedoraproject.FirewallD1` 这个就是firewalld注册的dbus interface了。

`dbus-send` 命令可以向 D-Bus消息总线发送消息并显示该消息的返回结果。有两个众所周知的消息总线：system bus（`Option -System`） 和每个用户session bus（ `-session`）。使用 `firewall-cmd` 也是通过 dbus interface 进行交互的。在使用`dbus-send` 时，必须指定其对应的消息接口 ` -dest`，该参数是连接到对应总线上的接口名称，以将消息发送到对应的dbus firewalld-server进行对应iptables规则的翻译。

现在有了dbus接口，需要了解改接口支持的方法 `methods`，属性 `properties `，信号`signals ` 等信息。

```
dbus-send --system --dest=org.fedoraproject.FirewallD1 --print-reply \
	/org/fedoraproject/FirewallD1 \
	org.freedesktop.DBus.Introspectable.Introspect
```

通过上述输出列出了通过防火墙 D-Bus 接口提供的所有方法、单一和属性。这是基于D-Bus DTD 的输出格式。所有 dbus服务都需要实现 `org.freedesktop.DBus.Introspectable.Introspect ` 方法。

知道了 方法 属性 信号，就可以直接对firewalld进行一个操作了。现在开始第一个例子。获取默认zone。

```
$ firewall-cmd --get-default-zone

dbus-send --system --dest=org.fedoraproject.FirewallD1 \
	--print-reply --type=method_call \ 
	/org/fedoraproject/FirewallD1 \
	org.fedoraproject.FirewallD1.getDefaultZone
```

**通过dbus接口来检索区域列表**

```
$ firewall-cmd --get-zones

dbus-send --system \
	--dest=org.fedoraproject.FirewallD1 \
	--print-reply --type=method_call \ 
	/org/fedoraproject/FirewallD1 \
	org.fedoraproject.FirewallD1.zone.getZones
```

**最常用的命令：查看当前zone所有策略**

```
$ firewall-cmd --zone=public --list-all

dbus-send --system \
	--dest=org.fedoraproject.FirewallD1 \
	--print-reply --type=method_call \
	/org/fedoraproject/FirewallD1 \
	org.fedoraproject.FirewallD1.getZoneSettings string:"public"
```

**获得inerface的properties** 

其实这里在命令行根本用不到，但是在封装时却会可以用到。

```
dbus-send --system \
	--print-reply --dest=org.fedoraproject.FirewallD1 \
	/org/fedoraproject/FirewallD1 \
	org.freedesktop.DBus.Properties.GetAll string:"org.fedoraproject.FirewallD1"
```

还可以通过其他的接口来查看对应的属性值

```
dbus-send --system --print-reply 
--dest=org.fedoraproject.FirewallD1 \
   /org/fedoraproject/FirewallD1 \
   org.freedesktop.DBus.Properties.Get \
   string:"org.fedoraproject.FirewallD1" \
   string:"version"


$ dbus-send --system --print-reply \
   --dest=org.fedoraproject.FirewallD1 \
   /org/fedoraproject/FirewallD1 org.freedesktop.DBus.Properties.Get \
   string:"org.fedoraproject.FirewallD1" \
   string:"interface_version"


$ dbus-send --system --print-reply \
   --dest=org.fedoraproject.FirewallD1 \
   /org/fedoraproject/FirewallD1 \
   org.freedesktop.DBus.Properties.Get \
   string:"org.fedoraproject.FirewallD1" \
   string:"state"

$ dbus-send --system --print-reply=literal \
   --dest=org.fedoraproject.FirewallD1 \
   /org/fedoraproject/FirewallD1 \
   org.freedesktop.DBus.Properties.Get \
   string:"org.fedoraproject.FirewallD1" \
   string:"state"
```

### 查询规则

**查询接口**

```
dbus-send --system \
    --dest=org.fedoraproject.FirewallD1 \
    --print-reply \
    --type=method_call \
    /org/fedoraproject/FirewallD1 \
    org.fedoraproject.FirewallD1.zone.getZoneOfInterface \
    string:"eth0"
```

**创建一个新zone**

```
dbus-send --session \
    --dest=org.freedesktop.DBus \
    --type=method_call \
    --print-reply /org/freedesktop/DBus  \
    org.fedoraproject.FirewallD1.config.addZone \
    string:"testapi"
```

**获得一个zone的所有规则（`zonesettings`）**

```
dbus-send --system \
    --dest=org.fedoraproject.FirewallD1  \
    --type=method_call \
    --print-reply /org/fedoraproject/FirewallD1  \
    org.fedoraproject.FirewallD1.getZoneSettings \
    string:"public"
```

**添加一个port**

```
dbus-send --system \
    --dest=org.fedoraproject.FirewallD1 \
    --print-reply --type=method_call \
    /org/fedoraproject/FirewallD1 \
    org.fedoraproject.FirewallD1.zone.addPort \
    string:"public" \
    string:"81" \
    string:"tcp" \
    uint64:300 
```



对应设置firewalld 面板所有属性的命令

```
firewall-cmd --zone=public --change-interface=eth0

firewall-cmd --zone=public --add-masquerade
firewall-cmd --zone=public --add-forward-port=port=1122:proto=tcp:toport=22:toaddr=192.168.100.3
firewall-cmd --zone=public --add-forward-port=port=1122:proto=tcp:toport=22:toaddr=10.0.0.3

firewall-cmd --add-protocol=tcp
firewall-cmd --add-protocol=udp

firewall-cmd --add-icmp-blocks=icmp
firewall-cmd --set-target=DROP

firewall-cmd --add-icmp-block=redirect
firewall-cmd --add-icmp-block=network-unknown

firewall-cmd --add-source-port=80/tcp
firewall-cmd --add-source-port=100/tcp

firewall-cmd --add-source=10.0.0.1
firewall-cmd --add-source=10.0.0.2

firewall-cmd --add-rich-rule='rule family=ipv4 source address=192.168.1.101/32 service name=telnet limit value=1/m accept'

firewall-cmd --add-icmp-block-inversion

firewall-cmd --new-zone=123 --permanen
```
### 执行远程命令



dbus接口支持远程命令的，通过dbus-send发送时，根据配置dbus的监听来完成远程的操作

```
DBUS_SESSION_BUS_ADDRESS=tcp:host=10.0.0.3,port=55557 
```

根据上述，参考加上官方文档，了解如何通过D-Bus接口操作FirewallD，虽然此处是使用了 `dbus-send`，但是也可以通过 qt 或者 其他的来管理 基于 dbus api的应用了。
