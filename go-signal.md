# go中的signal


## 什么是信号

在计算机科学中，信号是Unix、类Unix以及其他POSIX兼容的操作系统中进程间通讯的一种有限制的方式。它是一种异步的通知机制，用来提醒进程一个事件已经发生。

当一个信号发送给一个进程，操作系统中断了进程正常的控制流程，如果进程定义了对信号的处理，此时，程序将进入捕获到的信号对应的处理函数，否则执行默认的处理函数。

## Linux中信号的介绍

在Linux系统共定义了64种信号，分为两大类：**实时信号**与**非实时信号**，1-31为非实时，32-64种为实时信号。

> - 非实时信号： 也称为不可靠信号，为早期Linux所支持的信号，不支持排队，信号可能会丢失, 比如发送多次相同的信号, 进程只能收到一次. 信号值取值区间为1~31；
> - 实时信号： 也称为可靠信号，支持排队, 信号不会丢失, 发多少次, 就可以收到多少次. 信号值取值区间为32~64

Linux操作系统中，在终端上执行 `kill -l` 便可看到系统定义的所有信号

![](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/1380340-20201025201542398-333856774.png)

### 信号表

#### POSIX.1-1990标准信号

此表参考自：[POSIX信号](https://dsa.cs.tsinghua.edu.cn/oj/static/unix_signal.html)

| 信号    | 值       | 动作 | 说明                                                         |
| ------- | -------- | ---- | ------------------------------------------------------------ |
| SIGHUP  | 1        | Term | 终端控制进程结束(终端连接断开)                               |
| SIGINT  | 2        | Term | 用户发送INTR字符(Ctrl+C)触发                                 |
| SIGQUIT | 3        | Core | 用户发送QUIT字符(Ctrl+/)触发                                 |
| SIGILL  | 4        | Core | 非法指令(程序错误、试图执行数据段、栈溢出等)                 |
| SIGABRT | 6        | Core | 调用abort函数触发                                            |
| SIGFPE  | 8        | Core | 算术运行错误(浮点运算错误、除数为零等)                       |
| SIGKILL | 9        | Term | 无条件结束程序(不能被捕获、阻塞或忽略)                       |
| SIGSEGV | 11       | Core | 无效内存引用(试图访问不属于自己的内存空间、对只读内存空间进行写操作) |
| SIGPIPE | 13       | Term | 消息管道损坏(FIFO/Socket通信时，管道未打开而进行写操作)      |
| SIGALRM | 14       | Term | 时钟定时信号                                                 |
| SIGTERM | 15       | Term | 结束程序(可以被捕获、阻塞或忽略)                             |
| SIGUSR1 | 30,10,16 | Term | 用户保留                                                     |
| SIGUSR2 | 31,12,17 | Term | 用户保留                                                     |
| SIGCHLD | 20,17,18 | Ign  | 子进程结束(由父进程接收)                                     |
| SIGCONT | 19,18,25 | Cont | 继续执行已经停止的进程(不能被阻塞)                           |
| SIGSTOP | 17,19,23 | Stop | 停止进程(不能被捕获、阻塞或忽略)                             |
| SIGTSTP | 18,20,24 | Stop | 停止进程(可以被捕获、阻塞或忽略)                             |
| SIGTTIN | 21,21,26 | Stop | 后台程序从终端中读取数据时触发                               |
| SIGTTOU | 22,22,27 | Stop | 后台程序向终端中写数据时触发                                 |

更多的信号说明请查阅[man7](https://www.man7.org/linux/man-pages/man7/signal.7.html)

此表的操作为每个信号的默认配置，如下所示

| 动作 | 说明                                       |
| ---- | ------------------------------------------ |
| Term | 默认操作是，终止进程。                     |
| Ign  | 默认操作是，忽略信号。                     |
| Core | 默认操作是，终止该进程并核心转储           |
| Stop | 默认操作是，停止进程。                     |
| Cont | 默认操作是，如果当前已停止，则继续该进程。 |

## 信号的产生

信号是事件发生时对进程的通知机制。信号中断与硬件中断的相似之处在于打断了程序执行的正常流程。

信号事件的来源分为软件信号和硬件信号：

- 硬件信号： **用户输入**：比如在终端上按下组合键ctrl+C，产生SIGINT信号；**硬件异常**：CPU检测到内存非法访问等异常，通知内核生成相应信号，并发送给发生事件的进程；
- 软件信号： **通过系统调用**： 如，发送signal信号：`kill`，`raise`等。

## 发送的信号

- `Ctrl-C` 发送 INT signal (SIGINT)，通常导致进程结束
- `Ctrl-Z` 发送 TSTP signal (SIGTSTP); 通常导致进程挂起(suspend)
- `Ctrl-\` 发送 QUIT signal (SIGQUIT); 通常导致进程结束 和 dump core.

## 信号的处理

内核处理进程收到的signal是在当前进程的上下文，故进程必须是Running状态。当进程唤醒或者调度后获取CPU，则会从内核态转到用户态时检测是否有signal等待处理，处理完，进程会把相应的未决信号从链表中去掉。

signal信号处理时机： 内核 ==> 信号处理 ==> 用户

> 1. 内核态：在内核态，signal信号不起作用；
> 2. signal信号处理: 在用户态，signal所有未被屏蔽的信号都处理完毕；当屏蔽信号，取消屏蔽时，会在下一次内核转用户态的过程中执行；

### 信号处理方式

进程对信号的处理方式有3种： 

- 默认 接收到信号后按默认的行为处理该信号。 这种方式为多数应用采取的处理方式。
- 自定义处理 用自定义的信号处理函数来执行特定的动作
- 忽略忽略信号 接收到信号后不做任何反应。

对信号的处理动作：

- Term： 中止进程
- Ign： 忽略信号
- Core： 中止进程并保存内存信息
- Stop： 停止进程
- Cont： 继续运行进程

## Linux信号命令

### kill

kill命令用来终止指定的进程, 对于一个后台进程就须用kill命令来终止，我们就需要先使用ps/pidof/pstree/top等工具获取进程PID，然后使用kill命令来杀掉该进程。

> **命令格式**
> `kill[参数] [进程id]`

> **命令参数**
> `-l`  信号，若果不加信号的编号参数，则使用“-l”参数会列出全部的信号名称
> `-a`  当处理当前进程时，不限制命令名和进程号的对应关系
> `-p`  指定kill 命令只打印相关进程的进程号，而不发送任何信号
> `-s`  指定发送信号
> `-u`  指定用户

### killall

Linux系统中的`killall`用于杀死指定名字的进程（kill processes by name）。我们可以使用kill命令杀死指定进程PID的进程，如果要找到我们需要杀死的进程，我们还需要在之前使用ps等命令再配合grep来查找进程，而killall把这两个过程合二为一，是一个很好用的命令。

> **命令格式**
> `killall[参数] [进程名]`

> **命令参数**
> `-I`  忽略小写
> `-a`  当处理当前进程时，不限制命令名和进程号的对应关系
> `-i`  交互模式，杀死进程前先询问用户
> `-s`  发送指定的信号
> `-w`  等待进程死亡
> `-e`  要求匹配进程名称

### PKILL

`pkill` 与 `killall` 使用方法类似，用于杀死指定名称的进程

## Go语言中的Signal的使用

在Go语言中，处理信号仅需要3个步骤即可完成对信号的处理

- 信号的接收： `signalChan := make(chan os.Signal,1)`
- 信号的监听捕获： `signal.Notify(signalChan)`
- 信号的触发： `signal := <-signalChan`

注意事项：
- `SIGKILL kill -9`和`SIGSTOP kill -19` 信号可能不会被Notify方法捕获，因此无法处理这些信号。
- 如果在Notify方法中没有指定信号作为参数，那么该方法将捕获所有的信号。

## 在Go语言中的Signal的处理

在某些场景下，如，在大量并发及，批量处理未完成时，此时需要在Go程序中处理Signal信号，比如收到SIGTERM信号后优雅的关闭程序。

> 实例：在一个计算场景下，有5个goroutine在处理业务，当收到 `kill -15`时计算完成后退出程序， `kill -4`不做处理。

```go
package main

import (
	"fmt"
	"os"
	"os/signal"
	"sync"
	"syscall"
	"time"
)

var wg sync.WaitGroup

func exitProcess() {
	fmt.Println("等待进程完成")
	wg.Wait()
	fmt.Println("进程退出")
}

func process(n int) {
	i := n
	for {
		fmt.Println("process", n, ":", i)
		if i > 100 {
			break
		}
		time.Sleep(time.Second)
		i++
	}
	fmt.Println("process", n, "finnshed")
	defer wg.Done()
}

func main() {
	signals := make(chan os.Signal, 1)
	done := make(chan bool, 1)

	signal.Notify(signals, syscall.SIGILL, syscall.SIGTERM)

	go func() {
		for signal := range signals {
			switch signal {
			case syscall.SIGTERM, syscall.SIGQUIT:
				fmt.Println("kill -15 进程退出")
				exitProcess()
			case syscall.SIGILL:
				fmt.Println("kill -4")
			}
		}
		done <- true
	}()

	wg.Add(5)
	for n := 0; n < 10; n++ {
		go process(n)
	}

	fmt.Println("waiting signal...")
	wg.Wait()
	fmt.Println("exiting")
}
```

收到kill -4 信号打印kill -4

![](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/1380340-20201026001747221-1942598103.png)


收到kill -15 信号后，带程序处理完成后退出

![](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/1380340-20201026002133354-425851540.png)

![](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/1380340-20201026002146639-515404564.png)


