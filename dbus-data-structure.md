# dbus的数据结构


DBus中也是类似于静态语言，使用了“强类型”数据格式。在DBus上传递的所有数据都需要声明其对应的类型，下面整理了下，DBus中的数据类型，以及在DBus中声明的数据类型是什么意思。

| dbus类型  | 说明                                                         |
| --------- | ------------------------------------------------------------ |
| s         | string 字符串类型，可以声明 s:                               |
| a         | array 数组，可以声明为 a:                                    |
| v         | variant，variant:<type>:<value>                           |
| ()        | 结构体，声明时为双括号中间的为类型，可以是多个，例如(ss) 即这个结构体内包含两个字符串属性 |
| b         | 布尔值                                                       |
| SIGNATURE | signature类型                                                |
| y         | BYTE                                                         |
| d         | DOUBLE                                                       |
| t         | UINT64                                                       |
| x         | INT64                                                        |
| u         | UINT32                                                       |
| i         | INT32                                                        |
| q         | uint16                                                       |
| n         | INT16                                                        |
| {}        | 词典，这里声明为两个括号，中间为其对应的 key value，例如 {sv} 即 key是字符串类型，value是variant类型。 |
| o         | OBJECT_PATH 对象路径                                         |

- `a{sv}` :  是一个数组，为 一个键值对的词典，里面仅有一个

- `(ssssa{ss}as）` 为一个结构体， 里面属性有7个 两个词典（数组），五个字符串类型

- `(sssbsasa(ss)asba(ssss)asasasasa(ss)b)` 这个类型拆开为下：共16个属性

    ```
    (
     s string
     s string
     s string
     b bool
     s string
     as array only one string
     a(ss) two string type in the array
     as array only one string
     b bool
     a(ssss) four string type in the array
     as array only one string
     as array only one string
     as array only one string
     as array only one string
     a(ss)  two string type in the array
     b bool
     )
    ```
    对上述类型，python中就可以很灵活的声明
    ```
    ["", "", "", False, DEFAULT_ZONE_TARGET, [], [],
                             [], False, [], [], [], [], [], [], False]
    ```
    go 中就需要按照对应类型声明为不通的结构体，属性名称可以不为主，顺序需要一致。
    
    **其dbus收到的报文内容为**

    ```
       struct {
          string ""
          string "Public" 
          string "For use in public areas. You do not trust the other computers on networks to not harm your computer. Only selected incoming connections are accepted."
          boolean false
          string "default"
          array [
             string "ssh"
             string "dhcpv6-client"
          ]
          array [
             struct {
                string "55555-55557"
                string "tcp"
             }
          ]
          array [
             string "redirect"
             string "network-unknown"
          ]
          boolean true
          array [
             struct {
                string "1122"
                string "tcp"
                string "22"
                string "10.0.0.3"
             }
          ]
          array [
             string "eth0"
          ]
          array [
             string "10.0.0.1"
             string "10.0.0.2"
          ]
          array [
             string "rule family="ipv4" source address="192.168.1.101/32" service name="telnet" accept limit value="1/m""
          ]
          array [
             string "tcp"
             string "udp"
          ]
          array [
             struct {
                string "80"
                string "tcp"
             }
             struct {
                string "100"
                string "tcp"
             }
          ]
          boolean false
       }
    ```

- `ao`:  array，里面元素仅为一个`object_path`

## golang 中声明一个 Variant

在go中看到variant类型如下

```
type Variant struct {
	sig   Signature
	value interface{}
}
```

可以通过 `SignatureOf("short")`声明一个 `Signature`

然后在通过：`MakeVariantWithSignature(v interface{}, s Signature) Variant` 声明 对应的 Variant


***
注：其他数据类型与golang自己的数据类型一致，数组可以使用slice（类似php，python直接用数组替代即可更灵活）
***

> **More Reference**
>
> [dbus data type](http://www.fmddlmyy.cn/text54.html)
>
> [dbus data type conparision perl](http://www.fmddlmyy.cn/text54.html)

