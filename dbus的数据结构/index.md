# dbus的数据结构


DBus中也是类似于静态语言，使用了“强类型”数据格式。在DBus上传递的所有数据都需要声明其对应的类型，下面整理了下，DBus中的数据类型，以及在DBus中声明的数据类型是什么意思。

| dbus类型  | 说明                                                         |
| --------- | ------------------------------------------------------------ |
| s         | string 字符串类型，可以声明 s:                               |
| a         | array 数组，可以声明为 a:                                    |
| v         | variant，variant:&lt;type&gt;:&lt;value&gt;                           |
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
    [&#34;&#34;, &#34;&#34;, &#34;&#34;, False, DEFAULT_ZONE_TARGET, [], [],
                             [], False, [], [], [], [], [], [], False]
    ```
    go 中就需要按照对应类型声明为不通的结构体，属性名称可以不为主，顺序需要一致。
    
    **其dbus收到的报文内容为**

    ```
       struct {
          string &#34;&#34;
          string &#34;Public&#34; 
          string &#34;For use in public areas. You do not trust the other computers on networks to not harm your computer. Only selected incoming connections are accepted.&#34;
          boolean false
          string &#34;default&#34;
          array [
             string &#34;ssh&#34;
             string &#34;dhcpv6-client&#34;
          ]
          array [
             struct {
                string &#34;55555-55557&#34;
                string &#34;tcp&#34;
             }
          ]
          array [
             string &#34;redirect&#34;
             string &#34;network-unknown&#34;
          ]
          boolean true
          array [
             struct {
                string &#34;1122&#34;
                string &#34;tcp&#34;
                string &#34;22&#34;
                string &#34;10.0.0.3&#34;
             }
          ]
          array [
             string &#34;eth0&#34;
          ]
          array [
             string &#34;10.0.0.1&#34;
             string &#34;10.0.0.2&#34;
          ]
          array [
             string &#34;rule family=&#34;ipv4&#34; source address=&#34;192.168.1.101/32&#34; service name=&#34;telnet&#34; accept limit value=&#34;1/m&#34;&#34;
          ]
          array [
             string &#34;tcp&#34;
             string &#34;udp&#34;
          ]
          array [
             struct {
                string &#34;80&#34;
                string &#34;tcp&#34;
             }
             struct {
                string &#34;100&#34;
                string &#34;tcp&#34;
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

可以通过 `SignatureOf(&#34;short&#34;)`声明一个 `Signature`

然后在通过：`MakeVariantWithSignature(v interface{}, s Signature) Variant` 声明 对应的 Variant


***
注：其他数据类型与golang自己的数据类型一致，数组可以使用slice（类似php，python直接用数组替代即可更灵活）
***

&gt; **More Reference**
&gt;
&gt; [dbus data type](http://www.fmddlmyy.cn/text54.html)
&gt;
&gt; [dbus data type conparision perl](http://www.fmddlmyy.cn/text54.html)

