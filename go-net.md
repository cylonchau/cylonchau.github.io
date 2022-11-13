# go socket net包使用


Go语言标准库内建提供了net/http包，涵盖了HTTP客户端和服务端的具体实现。使用net/http包，我们可以很方便地编写HTTP客户端或服务端的程序。

## http服务端的创建流程

在使用http/net包创建服务端只需要两个步骤 绑定处理器函数 `func(ResponseWriter, *Request)`与 启用监听 `http.ListenAndServe`。

```go
package main

import &#34;net/http&#34;

func main() {
	http.HandleFunc(&#34;/&#34;, func(w http.ResponseWriter, r *http.Request) {
		w.Write([]byte(&#34;123&#34;))
	})
	http.ListenAndServe(&#34;:8080&#34;, nil)
}
```

通过分析`net/http`包中`server.go` 在执行创建http服务端主要执行了下面几个步骤： 
- `http.HandleFunc` 绑定处理函数
 - 所有的操作的方法都属于一个结构体 `ServeMux` 
   - m: 用户传入的路由和处理方法的映射表，路由和处理函数被定义为结构体`muxEntry`的属性
   - mu： 实例化出来的对象的读写锁
 - 调用`DefaultServeMux.Handle()`
 - 在`DefaultServeMux.Handle()`中调用`DefaultServeMux.HandleFunc(pattern, handler)`
 - 在将传入http.HandleFunc()的回调函数，与路由的映射信息，放到该`DefaultServeMux`的属性中 映射map中 `muxEntry`
- `http.ListenAndServe` 启动服务监听
 - 实例化一个server结构体
 - 调用 `ListenAndServe()`
 - `ListenAndServe()`中 `net.Listen(&#34;tcp&#34;, addr)` 启动tcp服务监听
 - Serve()中 appcet()处理用户连接，`go c.serve(connCtx)` 处理业务段（如判断信息，拼接http、找到对应处理函数）

综上所述，`net/http server.go` 一切的基础为ServeMux 和 Handler

Go语言的`net/http`包还封装了常用处理器，如 `FileServer`，`NotFoundHandler` `RedirectHandler`

## http客户端的使用

```go
package main

import (
	&#34;bytes&#34;
	&#34;fmt&#34;
	&#34;net/http&#34;
	&#34;reflect&#34;
)

func main() {
	resp, err := http.Get(&#34;http://www.baidu.com&#34;)

	if err != nil {
		fmt.Println(err)
		return
	}
	fmt.Println(reflect.TypeOf(resp.Body)) // *http.gzipReader
	b := bytes.NewBuffer(make([]byte, 1024))
	b.ReadFrom(resp.Body)
	fmt.Println(string(b.Bytes()))
}
```

post请求

```go
package main

import (
	&#34;net/http&#34;
	&#34;fmt&#34;
	&#34;io/ioutil&#34;
	&#34;net/url&#34;
)

func main() {

	postParam := url.Values{
		&#34;user&#34;:      {&#34;xxxxxx&#34;},
		&#34;Pwd&#34;: {&#34;1&#34;},
	}

	resp, err := http.PostForm(&#34;http://www.baidu.com/loginRegister/login&#34;, postParam)
	if err != nil {
		fmt.Println(err)
		return
	}

	defer resp.Body.Close()
	body, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		fmt.Println(err)
		return
	}

	fmt.Println(string(body))
}
```
