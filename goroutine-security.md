# go协程安全


## 多路复用

Go语言中提供了一个关键字`select`，通过`select`可以监听`channel`上的数据流动。select的用法与switch语法类似，由select开始一个新的选择块，每个选择条件由case语句来描述。只不过，select的`case`有比较多的限制，其中最大的一条限制就是每个case语句里必须是一个IO操作。

select 语法如下：

```go
   select {
    case <-chan1:
        // 如果chan1成功读到数据，则进行该case处理语句
    case chan2 <- 1:
        // 如果成功向chan2写入数据，则进行该case处理语句
    default:
        // 如果上面都没有成功，则进入default处理流程
    }
```

在一个select语句中，会按顺序从头至尾评估每一个发送和接收的语句；如果其中的任意一语句可以继续执行(即没有被阻塞)，那么就从那些可以执行的语句中任意选择一条来使用。如果没有任意一条语句可以执行(即所有的通道都被阻塞)，那么有两种可能的情况：⑴ 如果给出了default语句，那么就会执行default语句，同时程序的执行会从select语句后的语句中恢复。⑵ 如果没有default语句，那么select语句将被阻塞，直到至少有一个channel可以进行下去。

在一般的业务场景下，select不会用`default`，当监听的流中再没有数据，IO操作就 会阻塞现象，如果使用了`default`，此时可以出让CPU时间片。如果使用了`default` 就形成了非阻塞状态，形成了**忙轮训**，会占用CPU、系统资源。

阻塞与非阻塞使用场景

- 阻塞： 如：在监听超时退出时，如果100秒内无操作，择退出，此时添加了default会形成忙轮训，超时监听变成了无效。
- 非阻塞： 如，在一个只有一个业务逻辑处理时，主进程控制进程的退出。此时可以使用default。

## 定时器

Go语言中定时器的使用有三个方法
- `time.Sleep()`
- `time.NewTimer()` 返回一个时间的管道， time.C 读取管道的内容
- `time.After(5 * time.Second)` 封装了time.NewTimer()，反回了一个 `time.C`的管道

示例

```go
select {
    case <-time.After(time.Second * 10):
}
```
## 锁和条件变量

Go语言中为了解决协程间同步问题，提供了标准库代码，包`sync`和`sync/atomic`中。

### 互斥锁

互斥锁是传统并发编程对共享资源进行访问控制的主要手段，它由标准库sync中的Mutex结构体类型表示。sync.Mutex类型只有两个公开的指针方法，Lock和Unlock。Lock锁定当前的共享资源，Unlock进行解锁。

```go
package main

import (
	"fmt"
	"runtime"
	"sync"
	"time"
)

var mutex sync.Mutex

func print(str string) {
	mutex.Lock()         // 添加互斥锁
	defer mutex.Unlock() // 使用结束时解锁

	for _, data := range str { // 迭代器
		fmt.Printf("%c", data)
		time.Sleep(time.Second) // 放大协程竞争效果
	}
	fmt.Println()
}

func main() {
	go print("hello") // main 中传参
	go print("world")
	for {
		runtime.GC()
	}
}
```
![](https://img2020.cnblogs.com/blog/1380340/202010/1380340-20201026195343567-1267456404.png)


### 读写锁

读写锁的使用场景一般为**读多写少**，可以让多个读操作并发，同时读取，但是对于写操作是完全互斥的。也就是说，当一个goroutine进行写操作的时候，其他goroutine不能进行读写操作；当一个goroutine获取读锁之后，其他的goroutine获取写锁都会等待

![](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/1380340-20201026200352683-2138464590.png)

```go
package main

import (
	"fmt"
	"math/rand"
	"sync"
	"time"
)

var count int           // 全局变量count
var rwlock sync.RWMutex // 全局读写锁 rwlock

func read(n int) {
	for {
		rwlock.RLock()
		fmt.Printf("reading goroutine %d ...\n", n)
		num := count
		fmt.Printf("read goroutine %d finished，get number %d\n", n, num)
		rwlock.RUnlock()
	}

}
func write(n int) {
	for {
		rwlock.Lock()
		fmt.Printf("writing goroutine %d ...\n", n)
		num := rand.Intn(1000)
		count = num
		fmt.Printf("write goroutine %d finished，write number %d\n", n, num)
		rwlock.Unlock()
	}
}

func main() {
	for i := 0; i < 5; i++ {
		go read(i + 1)
		time.Sleep(time.Microsecond * 100)
	}
	for i := 0; i < 5; i++ {
		go write(i + 1)
		time.Sleep(time.Microsecond * 100)
	}
	for {

	}
}
```

![](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/1380340-20201026201220373-1468946475.png)

可以看出，读写锁控制下的多个写操作之间都是互斥的，并且写操作与读操作之间也都是互斥的。但是，多个读操作之间不存在互斥关系。

### Go语言中的死锁

死锁 `deadlock` 是指两个或两个以上的进程在执行过程中，由于竞争资源或者由于彼此通信而造成的一种阻塞的现象，若无外力作用，它们都将无法推进下去。此时称系统处于死锁状态或系统产生了死锁。

#### 单gorutine同时读写，写死锁

在一个gorutine中，当channel无缓冲，写阻塞，等待读取导致死锁

解决，应该至少在2个gorutine进行channle通讯，或者使用缓冲区。

```go
package main

func main() {
	channel := make(chan int)
	channel <- 1
	<-channel
}
```

#### 多gorutine使用一个channel通信，写先于读

代码顺序执行时，写操作阻塞，导致后面协程无法启动进行读操作，导致死锁

```go
package main

func main() {
	channel := make(chan int)
	channel <- 1
	go func() {
		<-channel
	}()
}
```

![](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/1380340-20201026193737321-444785180.png)

#### 多channel交叉死锁

在goroutine中，多个goroutine使用多个channel互相等待对方写入，导致死锁

```go
package main

func main() {

	channel1 := make(chan int)
	channel2 := make(chan int)

	go func() {

		select {
		case <-channel1:
			channel2 <- 1
		}
	}()

	select {
	case <-channel2:
		channel1 <- 1
	}
}
```
![](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/1380340-20201026194206902-684762536.png)

#### 隐性死锁

尽量不要将 互斥锁、读写锁 与 channel 混用情况下，让读先进行读时，因为没写入被阻塞，无法解除。写入时，因为没有读出被阻塞，锁无法解除，导致无数据输出，形成隐形死锁。此时编译器是不报错的。

![](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/1380340-20201026202657202-461749877.png)

```go
package main

import (
	"fmt"
	"sync"
)

func main() {
	channel := make(chan int)
	var rwlock sync.RWMutex
	go func() {
		for {
			rwlock.Lock()
			channel <- 1
			fmt.Println("write", 1)
			rwlock.Unlock()
		}
	}()

	go func() {
		for {
			rwlock.RLock()
			n := <-channel
			fmt.Println(n)
			rwlock.Unlock()
		}

	}()

	for {

	}
}
```

## Context 上下文

context定义了上下文类型，该类型在API边界之间以及进程之间传递截止时间，取消信号和其他请求范围的值。当在对请求传入一个上下文，可以选择将其替换为使用`WithCancel`，`WithDeadline`，`WithTimeout`。在取消后，从该context处派生的所有子请求也会被取消。


Context的结构体

- Deadline() 返回context的截止时间。
- Done() 返回一个channle，当timeout或cancelfuc将会close(chan)
- Err() 返回错误，未关闭Done()返回nil，取消，返回 `"context canceled"`, Deadline返回超时
- Value 返回值。

```go
type Context interface {
	// Deadline returns the time when work done on behalf of this context
	// should be canceled. Deadline returns ok==false when no deadline is
	// set. Successive calls to Deadline return the same results.
	Deadline() (deadline time.Time, ok bool)

	// Done returns a channel that's closed when work done on behalf of this
	// context should be canceled. Done may return nil if this context can
	// never be canceled. Successive calls to Done return the same value.
	// The close of the Done channel may happen asynchronously,
	// after the cancel function returns.
	//
	// WithCancel arranges for Done to be closed when cancel is called;
	// WithDeadline arranges for Done to be closed when the deadline
	// expires; WithTimeout arranges for Done to be closed when the timeout
	// elapses.
	//
	// Done is provided for use in select statements:
	//
	//  // Stream generates values with DoSomething and sends them to out
	//  // until DoSomething returns an error or ctx.Done is closed.
	//  func Stream(ctx context.Context, out chan<- Value) error {
	//  	for {
	//  		v, err := DoSomething(ctx)
	//  		if err != nil {
	//  			return err
	//  		}
	//  		select {
	//  		case <-ctx.Done():
	//  			return ctx.Err()
	//  		case out <- v:
	//  		}
	//  	}
	//  }
	//
	// See https://blog.golang.org/pipelines for more examples of how to use
	// a Done channel for cancellation.
	Done() <-chan struct{}

	// If Done is not yet closed, Err returns nil.
	// If Done is closed, Err returns a non-nil error explaining why:
	// Canceled if the context was canceled
	// or DeadlineExceeded if the context's deadline passed.
	// After Err returns a non-nil error, successive calls to Err return the same error.
	Err() error

	// Value returns the value associated with this context for key, or nil
	// if no value is associated with key. Successive calls to Value with
	// the same key returns the same result.
	//
	// Use context values only for request-scoped data that transits
	// processes and API boundaries, not for passing optional parameters to
	// functions.
	//
	// A key identifies a specific value in a Context. Functions that wish
	// to store values in Context typically allocate a key in a global
	// variable then use that key as the argument to context.WithValue and
	// Context.Value. A key can be any type that supports equality;
	// packages should define keys as an unexported type to avoid
	// collisions.
	//
	// Packages that define a Context key should provide type-safe accessors
	// for the values stored using that key:
	//
	// 	// Package user defines a User type that's stored in Contexts.
	// 	package user
	//
	// 	import "context"
	//
	// 	// User is the type of value stored in the Contexts.
	// 	type User struct {...}
	//
	// 	// key is an unexported type for keys defined in this package.
	// 	// This prevents collisions with keys defined in other packages.
	// 	type key int
	//
	// 	// userKey is the key for user.User values in Contexts. It is
	// 	// unexported; clients use user.NewContext and user.FromContext
	// 	// instead of using this key directly.
	// 	var userKey key
	//
	// 	// NewContext returns a new Context that carries value u.
	// 	func NewContext(ctx context.Context, u *User) context.Context {
	// 		return context.WithValue(ctx, userKey, u)
	// 	}
	//
	// 	// FromContext returns the User value stored in ctx, if any.
	// 	func FromContext(ctx context.Context) (*User, bool) {
	// 		u, ok := ctx.Value(userKey).(*User)
	// 		return u, ok
	// 	}
	Value(key interface{}) interface{}
}
```

演示使用可取消的上下文。可在函数结束时`defer cancel()` 防止`goroutine`的泄露。

```go
package main

import (
	"context"
	"fmt"
	"time"
)

func worker(ctx context.Context, name string) {
	n := 0
	for {
		select {
		case <-ctx.Done():
			fmt.Println(name, "去划水了", n)
			return
		default:
			fmt.Println(name, "干活中", n)
			time.Sleep(time.Second)
		}
		n++
	}
}

func main() {
	ctx, cancel := context.WithCancel(context.Background())

	for n := 0; n < 5; n++ {
		go worker(ctx, fmt.Sprintf("worker%d", n))
	}

	<-time.After(time.Second * 5)
	cancel()
	for {

	}
}
```

![](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/1380340-20201027003405631-971778476.png)

超时处理，`WithTimeout` 当时间到达设置的时间后退出，也可以使用`cancelFunc()`退出处理

```go
package main

import (
	"context"
	"fmt"
	"time"
)

func worker(ctx context.Context, name string) {
	n := 0
	for {
		select {
		case <-ctx.Done():
			fmt.Println(name, "去划水了", n)
			return
		default:
			fmt.Println(name, "干活中", n)
			time.Sleep(time.Second)
		}
		n++
	}
}

func main() {
	//ctx, cancel := context.WithCancel(context.Background())
	ctx, cancel := context.WithTimeout(context.Background(), 3*time.Second)

	for n := 0; n < 2; n++ {
		go worker(ctx, fmt.Sprintf("worker%d", n))
	}

	<-time.After(time.Second * 5)
	fmt.Println("取消了")
	cancel()
}
```
![](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/1380340-20201027003955469-353595368.png)

WithDeadline，在标准库中可以看出，实际上WithTimeout是封装了WithDeadline。其功能也是超时退出。

![](https://img2020.cnblogs.com/blog/1380340/202010/1380340-20201027004554454-1396510619.png)

```go
package main

import (
	"context"
	"fmt"
	"time"
)

func worker(ctx context.Context, name string) {
	n := 0
	for {
		select {
		case <-ctx.Done():
			fmt.Println(name, "去划水了", n)
			fmt.Println(ctx.Err())
			return
		default:
			fmt.Println(name, "干活中", n)
			time.Sleep(time.Second)
		}
		n++
	}
}

func main() {
	ctx, cancel := context.WithDeadline(context.Background(), time.Now().Add(time.Second*3))
	for n := 0; n < 2; n++ {
		go worker(ctx, fmt.Sprintf("worker%d", n))
	}

	<-time.After(time.Second * 5)
	fmt.Println("取消了")
	cancel()
}
```
![](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/1380340-20201027004659406-744263091.png)

### Context总结

- Context是Go语言在1.7中加入标准库的，是作为`Goroutine`线程安全，防止线程泄露的上下文管理的操作。
- context包的核心是`Context`结构体。
- Context的常用方法为 `WithTimeout()` 与 `WithCancel()`
- Context在使用时，不要放在结构体内使用，要以函数的参数进行传递。
- Context是线程安全的，可以在多个`Goroutine`传递，当对其取消操作时，所有`Goroutine`都执行取消操作。
