# go socket TCP协议实现


在TCP/IP协议中，“IP地址&#43;TCP或UDP端口号”唯一标识网络通讯中的一个进程。“IP地址&#43;端口号”就对应一个socket。欲建立连接的两个进程各自有一个socket来标识，那么这两个socket组成的socket pair就唯一标识一个连接。因此可以用Socket来描述网络连接的一对一关系。

常用的Socket类型有两种：流式Socket（SOCK_STREAM）和数据报式Socket（SOCK_DGRAM）。流式是一种面向连接的Socket，针对于面向连接的TCP服务应用；数据报式Socket是一种无连接的Socket，对应于无连接的UDP服务应用。

套接字通讯原理示意

![](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/1380340-20201015112932245-832922409.png)

## TCP的C/S架构

![](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/1380340-20201015112953158-858280748.png)

在整个通信过程中，服务器端有两个socket参与进来，但用于通信的只有conn这个socket。它是由 listener创建的。隶属于服务器端。客户端有一个socket参与进来。

`net.Listen()` 建立一个用于连接监听的套接字
`listen.Accept()` // 阻塞监听客户端连接请求，成功用于连接，返回用于通信的socket
`net.Dial()` 客户端向服务端发起连接建立一个socket连接

![](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/1380340-20201015113232583-2036839633.png)

## 并发的C/S模型通信

### Server

Accept()函数的作用是等待客户端的链接，如果客户端没有链接，该方法会阻塞。如果有客户端链接，那么该方法返回一个Socket负责与客户端进行通信。所以，每来一个客户端，该方法就应该返回一个Socket与其通信，因此，可以使用一个死循环，将Accept()调用过程包裹起来。

需要注意，实现并发处理多个客户端数据的服务器，就需要针对每一个客户端连接，单独产生一个Socket，并创建一个单独的goroutine与之完成通信。

```go
package main

import (
	&#34;fmt&#34;
	&#34;net&#34;
	&#34;strings&#34;
)

func handleConnect(conn net.Conn){
	var (
		b []byte
		err error
		n int
	)
	fmt.Println(conn.RemoteAddr(),&#34;建立连接.&#34;)
	defer conn.Close()
	b = make([]byte,4096)
        // 客户端可能持续不断的发送数据，因此接收数据的过程可以放在for循环中，服务端也持续不断的向客户端返回处理后的数据。
	for {
		n,err = conn.Read(b)

		content := strings.Trim(string(b[:n]),&#34;\r\n&#34;) // window中传送的内容存在换行符，作为判断时需要删除
                // 当客户端退出，服务端从chan中读取内容时是没有的，因此的到0 或者客户端主动退出输入exit或者quit
		if n == 0 || content == &#34;exit&#34; || content == &#34;quit&#34; {
			fmt.Println(&#34;客户端退出：&#34;,conn.RemoteAddr())
			return
		}

		if err != nil {
			fmt.Println(err)
			return
		}

		if _,err =  conn.Write([]byte(fmt.Sprintf(&#34;server reply:%s&#34;,b[:n])));err !=nil {
			fmt.Println(err)
			return
		}
		fmt.Println(&#34;client send: &#34;,content)
	}
}

func main() {
	var (
		listener net.Listener
		err      error
		conn     net.Conn
	)
	// 建立一个用于连接监听的套接字
	if listener, err = net.Listen(&#34;tcp&#34;, &#34;10.0.0.1:8088&#34;); err != nil {
		fmt.Println(err)
		return
	}
	defer listener.Close()

	fmt.Println(&#34;waiting client connect.&#34;)

	// 阻塞监听客户端连接请求，成功用于连接，返回用于通信的socket
	for {
		if conn, err = listener.Accept(); err != nil {
			fmt.Println(err)
			return
		}

		go handleConnect(conn)
	}
}
```

使用nc作为客户端向服务端发送信息

![](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/1380340-20201015113907465-654602281.png)

### 自定义客户端

客户端需要持续的向服务端发送数据，同时也要接收从服务端返回的数据。因此可将发送和接收放到不同的协程中。
- 主协程循环接收服务器回发的数据（该数据应已转换为大写），并打印至屏幕；
- 子协程循环从键盘读取用户输入数据。
- 读取键盘输入可使用 `os.Stdin.Read()`。

注意事项：
- 服务端有对 exit返回的是 `io.EOF`
- 当服务端断开时，chan读取的信息就为0了即服务端已经退出，如果客户端不退出会一直报错

```go
package main

import (
	&#34;fmt&#34;
	&#34;io&#34;
	&#34;net&#34;
	&#34;os&#34;
	&#34;strings&#34;
)

func main() {

	var (
		conn net.Conn
		err  error
		n    int
	)

	if conn, err = net.Dial(&#34;tcp&#34;, &#34;10.0.0.1:8088&#34;); err != nil {
		fmt.Println(err, 111)
		return
	}
	defer conn.Close()

	go func() {
		str := make([]byte, 1024)
		for {
			n, err := os.Stdin.Read(str)
			content := strings.ToLower(strings.Trim(string(str[:n]), &#34;\r\n&#34;))

			if n == 0 {
				fmt.Println(&#34;与服务端断开连接&#34;)
				return
			}

			if err == io.EOF || content == &#34;quit&#34; {
				return
			}

			if err != nil {
				fmt.Println(1, err)
				continue
			}

			_, err = conn.Write([]byte(content))
			if err != nil {
				fmt.Println(111, err)
				return
			}

		}
	}()

	byt := make([]byte, 1024)
	for {
		if _, err = conn.Read(byt); err != nil {
			if err == io.EOF {
				return
			}
			fmt.Println(err)
			continue
		}
		fmt.Println(&#34;server reply:&#34;, string(byt[:n]))
	}
}
```

![](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/1380340-20201015115233290-1311572382.png)
