# dbus客户端使用指南


DBus是Linux使用的进程间通信机制，允许各个进程互相访问，而不需要为每个其他组件实现自定义代码。即使对于系统管理员来说，这也是一个相当深奥的主题，但它确实有助于解释linux的另一部分是如何工作的。

这里主要介绍 `dbus-send` 与 `GDbus` cli工具，其他的还有`QtDbus` , `d-feet`...

命令行工具`dbus-send` ，是freedesktoop提供的dbus包配套的命令客户端工具，可用于发送dbus消息。

`GDbus` GLib实现的dbus工具。较与 `dbus-send`，拥有更完整的功能。

`d-feet`: 可以处理所有D-Bus服务的GUI应用程序。

## dbus-send

dbus有两种消息总线 （`message bus`）：`system bus` 和 `session bus`，通过使用 `--system`和 `--session` 选项来通过`dbus-send` 向系统总线或会话总线发送消息。如果两者都未指定，默认为**session bus*.

借此，顺道聊下 `system bus` 和 `session bus`：

- `System Bus`:
  - 在桌面上，为所有用户提供一条总线.
  - 专用于系统服务。
  - 有关于低级时间，例如 网络连接，USB设备。
  - 在嵌入式Linux系统中，system bus是唯一D-Bus类型。
  - 

- `Session Bus`:
  - 每个用户会话一个实例
  - 为用户应用提供那个桌面服务。
  - 连接到 `X-session`

### 参数选项

`--dest=NAME` : 这个是必选的参数，指定要接收消息的接口名称。例如 `org.freedesktop.ExampleName` 。

`--print-reply`:  打印回复消息。

`--print-reply=literal`: 如选项一样，打印回复正文。如有特殊字符，如对象或 object 则按字面打印，没有标点符号、转义字符等。

`--reply-timeout=` : 可选参数，等待回复的超时时长，单位为 毫秒。

`--system|--session`:  发送的消息是*system bus*还是*session bus*，默认为 *session bus*.

`--type=method_call|signal`: 调用的方法：默认为signal。

必须始终指定要发送的消息的对象路径和名称。以下参数（如果有）是消息内容（消息参数）。这些值作为类型指定的值给出，可能包括如下所述的容器（数组、dict和变体）。

### 支持参数

`dbus-send` 发送的消息，在调用方法需要传参数时，必须将这些值给出。`dbus-send` 支持传入的参数的类型，并不为D-Bus支持的所有的数据类型，仅为一些简单的类型：如

- Type: 这里`type` 仅仅为简单的数据类型，即 `type:content`  ,支持的内容如下： `string | int16 | uint16 | int32 | uint32 | int64 | uint64 | double | byte | boolean | objpath`。
- 数组：`array = array:&lt;type&gt;:&lt;value&gt;[,&lt;value&gt;...]`
- 词典: `dict = dict:&lt;type&gt;:&lt;type&gt;:&lt;key&gt;,&lt;value&gt;[,&lt;key&gt;,&lt;value&gt;...]`。
- 变体：`variant = variant:&lt;type&gt;:&lt;value&gt;`。

根据官网的解析出来后如上述集中数据类型，更详细的描述可以根据官方 [dbus-send](https://dbus.freedesktop.org/doc/dbus-send.1.html) 进行参考。

**可以通过一张图来理解 `dbus-send` 发送一个消息所需的几个必须参数**

![](https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/1380340-20211011223436036-907208415.png)


**通过简单的命令，来了解一个 `dbus-send` 命令如何传入参数**

```bash
 dbus-send --dest=org.freedesktop.ExampleName \  # service
   /org/freedesktop/sample/object/name \  # object
   org.freedesktop.ExampleInterface.ExampleMethod \ # interface.method
   int32:47 string:&#39;hello world&#39; double:65.32	\ # param int
   array:string:&#34;1st item&#34;,&#34;next item&#34;,&#34;last item&#34; \ # param array
   dict:string:int32:&#34;one&#34;,1,&#34;two&#34;,2,&#34;three&#34;,3 \ # param dict
   variant:int32:-8 \ # param variant
   objpath:/org/freedesktop/sample/object/name # param object_path
```

### 使用案例

如列出所有总线接口

```shell
dbus-send --session \
  --dest=org.freedesktop.DBus \
  --type=method_call \
  --print-reply \
  /org/freedesktop/DBus \
  org.freedesktop.DBus.ListNames
```

查看对方总线所支持的对象接口，`org.freedesktop.DBus.Introspectable` 、`org.freedesktop.DBus.Properties` 和 `org.freedesktop.PowerManagement`。每个接口实现一些方法和信号。这些是你可以与之互动的东西。

```bash
dbus-send --session \
	--type=method_call \
	--print-reply \
	--dest=org.freedesktop.DBus \ 
	/ \
	org.freedesktop.DBus.Introspectable.Introspect
```

`dbus-send`，也支持调用远程总线接口，通过默认通过 `DBUS_SESSION_BUS_ADDRESS` 或 `DBUS_SYSTEM_BUS_ADDRESS`，来指定远程的总线。

```
DBUS_SESSION_BUS_ADDRESS=&#34;&#34;
dbus-send --session \
	--type=method_call \
	--print-reply \
	--dest=org.freedesktop.DBus \ 
	/ \
	org.freedesktop.DBus.Introspectable.Introspect
```

## gdbus

gdbus是 GLib实现的dbus工具。较与 `dbus-send`，拥有更完整的功能。

`introspect` : 可以打印出对象的接口和属性值。对应对象的所有者需要实现`org.freedesktop.DBus.Introspectable` 的接口。使用 `--xml`选项，将打印返回的`xml ` 格式。`--recurse` 选项可将其子级等打印，`--only` 选项仅打印具有属性的接口。

`monitor`: 类似于 `dbus-monitor`

`call`: 调用一个方法，传入的必须为 `GVariant` ,而相应的也为`GVariant`。

`emit`: 发出信号。信号中包含的每个参数除字符串外都必须序列化为GVariant。

使用案例

```
gdbus introspect --system \
	--dest org.freedesktop.UPower \
	--object-path \
	/ \
	--recurse  \
	--only-properties 
```

通过call 来向一个dbus service发送信息

```
gdbus call --session \
             --dest org.freedesktop.Notifications \
             --object-path /org/freedesktop/Notifications \
             --method org.freedesktop.Notifications.Notify \
             my_app_name \
             42 \
             gtk-dialog-info \
             &#34;The Summary&#34; \
             &#34;Here&#39;s the body of the notification&#34; \
             [] \
             {} \
             5000
(uint32 12,)
```

监听一个服务的对象

```
gdbus monitor \
	--system \
	--dest org.freedesktop.NetworkManager \
	--object-path /org/freedesktop/NetworkManager/AccessPoint/4141
```

发送信号

```
gdbus emit --session \
	--object-path /foo \
	--signal org.bar.Foo &#34;[&#39;foo&#39;, &#39;bar&#39;, &#39;baz&#39;]&#34;
```

想特定进程发送信号，`--dest 为指定进程。

```
 gdbus emit \
 	--session \
 	--object-path /bar \
 	--signal org.bar.Bar someString \
 	--dest :1.42
```
