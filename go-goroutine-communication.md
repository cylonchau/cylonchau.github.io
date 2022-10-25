# 协程通讯

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;channel是Go语言中的一个核心**数据类型**，channel是一个数据类型，主要用来解决协程的同步问题以及协程之间数据共享（数据传递）的问题。在并发核心单元通过它就可以发送或者接收数据进行通讯，这在一定程度上又进一步降低了编程的难度。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;goroutine运行在相同的内存地址空间，channel可以避开所有内存共享导致的坑；通道的通信方式保证了同步性。数据通过channel：同一时间只有一个协程可以访问数据：所以不会出现数据竞争，确保并发安全。

## channel的定义

channel是对应make创建的底层数据结构的引用。 创建语法： `make(chan Type, capacity)`

```go
channel := make(chan bool) //创建一个无缓冲的bool型Channel ，等价于make(chan Type, 0)
channel := make(chan bool, 1024) //创建一个有缓冲，切缓冲区为1024的bool型Channel 

channel <- x           //向一个Channel发送一个值
<- channel             //从一个Channel中接收一个值
x = <- channel         //从Channel c接收一个值并将其存储到x中
x, ok = <- channel     //从Channel接收一个值，如果channel关闭了或没有数据，那么ok将被置为false
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;channel是一个引用类型，当复制一个channel或用于函数参数传递时，我们只是拷贝了一个channel引用，因此调用者和被调用者将引用同一个channel对象。和其它的引用类型一样，channel的零值（定义未初始化）也是nil。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在默认情况下，channel**接收**和**发送**数据都是阻塞的，（`channel <- 1`，写端写数据，读端不在读。写端阻塞； `str := <-channel` 读端读数据， 同时写端不在写，读端阻塞。）除非另一端已经准备好，这样就使得goroutine同步变的更加的简单，而不需要显式的lock。

![](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/1380340-20201023210604900-1402843122.png)


**示例**

```go
package main

import (
	"fmt"
	"runtime"
	"time"
)

var c = make(chan int32)

func printstr(s string) {
	for _, value := range s {
		fmt.Printf("写入%+q\r\n", value)
		time.Sleep(time.Second)
		c <- value
	}
}

func main() {
	runtime.GOMAXPROCS(1)
	go func() {
		time.Sleep(time.Second)
		printstr("hello")
	}()

	go func() {
		for v := range c {
			fmt.Printf("读取%+q\r\n", v)
		}
	}()
	for {
              ;
	}
}
```
![](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/1380340-20201023205925898-1792789339.png)

## channel的缓冲

### 无缓冲的channel

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;无缓冲的channel `unbuffered channel` 是指在接收前没有能力保存任何值的通道。这种类型的channel **要求发送端和接收端同时准备好**，才能完成发送和接收操作。否则，通道会导致先执行发送或接收操作的**阻塞等待**。顾又称为同步通信

- 阻塞：由于某种原因数据没有到达，当前协程（线程）持续处于等待状态，直到条件满足，才接触阻塞。
- 同步：在两个或多个协程（线程）间，保持数据内容一致性的机制。

示例如上，写了没有读会导致阻塞，读了没有写会导致堵塞

### 有缓冲的channel

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;有缓冲的通道（`buffered channel`）是一种在被接收前能存储一个或者多个数据值的通道。这种类型的channel并不强制要求`goroutine`之间必须同时完成发送和接收。通道会阻塞发送和接收动作的条件也不同。

- 只有channel通道中没有要接收的值时，接收动作才会阻塞。
- 只有通道没有可用缓冲区容纳被写入（发送）的值时，发送动作才会阻塞。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;有缓冲的channel和无缓冲的channel之间的不同：无缓冲的channel保证进行发送和接收的 `goroutine` 会在同一时间进行数据交换；有缓冲的channel没有这种保证。

![](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/1380340-20201023210556108-1125304418.png)

示例

```go
package main

import (
	"fmt"
	"runtime"
	"time"
)

var c = make(chan int32, 10)

func printstr(s string) {
	for _, value := range s {
		fmt.Printf("写入%+q\r\n", value)
		c <- value
	}
}

func main() {
	runtime.GOMAXPROCS(1)
	go func() {

		printstr("hello")
	}()

	go func() {

		time.Sleep(time.Second * 2)
		fmt.Println("读通道开始读取数据")
		for v := range c {
			fmt.Printf("读取%+q\r\n", v)
		}
	}()
	for {

	}
}
```

![](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/1380340-20201023210810282-1509130297.png)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;结果可以看出，如果给定了一个缓冲区容量，channel就是**异步**的。只要缓冲区有未使用空间用于发送数据，或还包含可以接收的数据，那么其通信就会**无阻塞**地进行。

### channel的关闭

当发送的一端没有更多的数据发送到channel的话，需要使接收端也能及时知道channel中没有多余的数据可以接收。因此可以通过 `close()`函数来关闭channel的实现。

**示例**

```go
package main

import (
	"fmt"
	"runtime"
	"time"
)

var c = make(chan int32, 10)

func printstr(s string) {
	for _, value := range s {
		fmt.Printf("写入%+q\r\n", value)
		c <- value
	}
	close(c)
}

func main() {
	runtime.GOMAXPROCS(1)
	go func() {
		printstr("hello")
	}()

	time.Sleep(time.Second * 2)
	fmt.Println("读通道开始读取数据")
	for {
		if char, ok := <-c; ok {
			fmt.Printf("读取%+q\r\n", char)
		} else {
			break
		}
	}
}
```
![](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/1380340-20201023211728625-1454348143.png)

> **提示**
> - channel不像文件一样需要经常去关闭，只有当你确实没有任何发送数据了，或者你想显式的结束range循环之类的，才去关闭channel；
> - 关闭channel后，无法向channel 再发送数据(引发 panic 错误后导致接收立即返回零值)；
> - 关闭channel后，可以继续从channel接收数据（读取到的数据为channel类型的默认值，如int默认值0 string默认值""）；
> - 对于nil channel，无论收发都会被阻塞。

###  缓冲channel 和 非缓冲channel的区别

- 缓冲channel的创建方式为`make(chan TYPE,CAPCTIY)`，非缓冲channel的创建方式为`make(chan TYPE)`
- 缓冲channel的通信方式为同步通信，非缓冲channel的通信方式为异步通信

## 单项channel及应用

默认情况下，channel是双向的，既可以往里面发送数据也可以接收数据。但是，常将channel作为参数进行传递而只希望对方是单向使用的，要么只让它发送数据，要么只让它接收数据，这时候可以指定通道的方向。

### 单项channel的声明

- 双向channel `ch = make(chan int)`
- 单向写channel: `var ch chan <- int` `ch = make(chan <- int)`
- 单向读channel:	`var ch <- chan int` `ch = make(<-chan int)`

可以将 channel 隐式转换为单向队列，只收或只发，不能将单向 channel 转换为普通 channel，示例：

```go
package main

import (
	"fmt"
	"runtime"
)

var c = make(chan string, 10)

func read(c <-chan string) {
	fmt.Println("读通道开始读取数据")
	for {
		if char, ok := <-c; ok {
			fmt.Printf("读取%s\r\n", char)
		} else {
			break
		}
	}
}

func write(ch chan<- string, str []string) {
	defer close(ch)
	for _, value := range str {
		fmt.Printf("写入%+q\r\n", value)
		ch <- value
	}
}

func main() {
	runtime.GOMAXPROCS(3)

	go write(c, []string{"h", "e", "l", "l", "o"})
	read(c)
}
```
![](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/1380340-20201023233734222-1224564313.png)
