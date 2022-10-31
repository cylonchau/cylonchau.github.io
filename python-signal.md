# 

## 什么是信号

信号（signal）-- 进程间通讯的一种方式，也可作为一种软件中断的方法。一个进程一旦接收到信号就会打断原来的程序执行来按照信号进行处理。

简化术语，信号是一个事件，用于中断运行功能的执行。信号始终在主Python线程中执行。对于信号，这里不做详细介绍。

Python封装了操作系统的信号功能的库 `singal` 的库。`singal`  库可以使我们在python程序中中实现信号机制。

https://zh.wikipedia.org/wiki/Unix%E4%BF%A1%E5%8F%B7)

## Python的信号处理



首先需要了解Python为什么要提供 `signal Library`。信号库使我们能够使用信号处理程序，以便当接收信号时都可以执行自定义任务。

>Mission：当接收到信号时执行信号处理方法

可以通过使用 `signal.singal()` 函数来实现此功能

### Python对信号的处理



通常情况下Python 信号处理程序总是会在主 Python 主解析器的主线程中执行，即使信号是在另一个线程中接收的。 这意味着信号不能被用作线程间通信的手段。 你可以改用 [`threading`](https://docs.python.org/zh-cn/3/library/threading.html#module-threading) 模块中的同步原语。



Python信号处理流程，需要对信号处理程序（signal handling ）简要说明。`signal handling ` 是一个任务或程序，当检测到特定信号时，处理函数需要两个参数，即信号id `signal number` （Linux 中 1-64），与堆栈帧 `frame`。通过相应信号启动对应 `signal handling` ，`signal.signal()` 将为信号分配 处理函数。

如：当运行一个脚本时，取消，此时是捕获到一个信号，可以通过捕获信号方式对程序进行异步的优雅处理。通过将信号处理程序注册到应用程序中：

```py
import signal  
import time  

def handler(a, b):  # 定义一个signal handling
    print("Signal Number:", a, " Frame: ", b)  
  
signal.signal(signal.SIGINT, handler)  # 将handle分配给对应信号
  
while True:  
    print("Press ctrl + c")
    time.sleep(10) 
```

![python signal processing, python SIGINT handler](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/python-signal-processing.png)

如果不对对应信号进行捕获处理时，python将会抛出异常。

```
root@Seal:/mnt/d/pywork/signal# python signal.py
^CTraceback (most recent call last):
  File "signal.py", line 3, in <module>
    while True:
KeyboardInterrupt
```

## 信号枚举

信号的表现为一个int，Python的信号库有对应的信号枚举成员

其中常用的一般有，

SIGINT  control+c

SIGTERM  终止进程 软件终止信号

SIGKILL  终止进程 杀死进程

SIGALRM 超时

| 信号                                               | 说明                                                         |
| -------------------------------------------------- | ------------------------------------------------------------ |
| **SIG_DFL**                                        |                                                              |
| **SIG_IGN**                                        | 标准信号处理程序，它将简单地忽略给定的信号                   |
| **SIGABRT** <br>**SIGIOT**                         | 来自 abort 的中止信号。<br>abort 导致异常进程终止。通常由检测内部错误或严重破坏约束的库函数调用。例如，如果堆的内部结构被堆溢出损坏，`malloc()`将调用`abort()` |
| **SIGALRM**<br/>**SIGVTALRM** **<br>** **SIGPROF** | 如果你用 setitimer 这一类的报警设置函数设置了一个时限，到达时限时进程会接收到 SIGALRM, SIGVTALRM 或者 SIGPROF。但是这三个信号量的含义各有不同，SIGALRM 计时的是真实时间，SIGVTALRM计时的是进程使用了多少CPU时间，而 SIGPROF 计时的是进程和代表该进程的内核用了多少时间。 |
| **SIGBUS**                                         | 总线发生错误时，进程接收到一个SIGBUS信号。举例来说，存储器访问对齐或者或不存在对应的物理地址都会产生SIGBUS信号。 |
| **SIGCHLD**                                        | 当子进程终止、被中断或被中断后恢复时，SIGCHLD信号被发送到进程。该信号的一个常见用法是指示操作系统在子进程终止后清理其使用的资源，而不显式调用等待系统调用。 |
| **SIGILL**                                         | 非法指令。当进程试图执行非法、格式错误、未知或特权指令时，SIGILL信号被发送到该进程。 |
| **SIGKILL**                                        | 发送SIGKILL信号到一个进程可以使其立即终止(KILL)。与SIGTERM和SIGINT相不同的是，这个信号不能被捕获或忽略，接收过程在接收到这个信号时不能执行任何清理。 以下例外情况适用: |
| **SIGINT**                                         | 来自键盘的中断 (CTRL + C)。`KeyboardInterrupt`               |
| **SIGPIPE**                                        | 当一个进程试图写入一个没有连接到另一端进程的管道时，SIGPIPE信号会被发送到该进程。 |
| **SIGTERM **                                       | 终结信号。 KILL -15 \|KILL                                   |
| **SIGUSR1**<br>**SIGUSR2**                         | 用户自定义信号                                               |
| **SIGWINCH**                                       | 终端窗口大小已变化                                           |
| **SIGHUP**                                         | 在控制终端上检测到挂起或控制进程的终止。                     |

Reference：[signal-wikipedia](

## 信号函数

Python的信号库中也有很多常用的函数

### signal.alarm(time)

创建一个 `SIGALRM` 类型的信号，time为预定的时间，设置为0时取消先前设置的定时器

### **signal.pause()**

可以使代码逻辑处理过程睡眠，直到收到信号，然后调用对应的handler。

```python
import signal
import os
import time

def do_exit(sig, stack):
    raise SystemExit('Exiting')

signal.signal(signal.SIGINT, signal.SIG_IGN)
signal.signal(signal.SIGUSR1, do_exit)

print('My PID:', os.getpid())

signal.pause()
```

在执行时，忽略了ctrl + c的信号，对USR1做退出操作

### **signal.setitimer(which, seconds, interval)**

which： `signal.ITIMER_REAL，`[`signal.ITIMER_VIRTUAL`](https://docs.python.org/2/library/signal.html?highlight=signal#signal.ITIMER_VIRTUAL) 或 `signal.ITIMER_PROF`

seconds：多少秒后触发which。seconds设置为0可以清除which的计时器。

interval：每隔interval秒后触发一次

### os.getpid()

获得当前执行程序的pid

## Windows下信号的使用

在Linux中，可以通过任何可接受的信号枚举值作为信号函数的参数。在Windows中，`SIGABRT`, `SIGFPE`, `SIGINT`, `SIGILL`, `SIGSEGV`, `SIGTERM`, `SIGBREAK`。

## 当signal handling需要参数怎么办

在一些时候，signal handling的操作需要对应主进程传递进来一些函数，而在整个项目中执行过程中的变量与 signal handling不处于一个作用域中，而`signal.signal()` 不能传递其他的参数，这个时候可以使用 `partial` 创建一个闭包来解决这个问题。

例如：

```python
import signal
import os
import sys
import time

from functools import partial

"""
这里signal frame默认参数需要放到最后
"""
def signal_handler(test_parameter1, test_parameter2, signal_num, frame):
    print "signal {} exit. {} {}".format(signal_num, test_parameter1, test_parameter2)
    sys.exit(1)


a=1
b=2
signal.signal(signal.SIGINT, partial(signal_handler, a, b) )
print('My PID:', os.getpid())

signal.pause()
```

## 忽略信号

signal定义了忽略接收信号的方法。为了实现信号的处理，需要使用`signal.signal()` 将默认的信号与`signal.SIG_IGN` 注册，即可忽略对应的信号中断，`kill -9` 不可忽略 。

```python
import signal
import os
import time

def receiveSignal(signalNumber, frame):
    print('Received:', signalNumber)
    raise SystemExit('Exiting')
    return

if __name__ == '__main__':
    # register the signal to be caught
    signal.signal(signal.SIGUSR1, receiveSignal)

    # register the signal to be ignored
    signal.signal(signal.SIGINT, signal.SIG_IGN)

    # output current process id
    print('My PID is:', os.getpid())

    signal.pause()
```

## 常用的信号

```python
import signal
import os
import time
import sys

def readConfiguration(signalNumber, frame):
    print ('(SIGHUP) reading configuration')
    return

def terminateProcess(signalNumber, frame):
    print ('(SIGTERM) terminating the process')
    sys.exit()

def receiveSignal(signalNumber, frame):
    print('Received:', signalNumber)
    return
	
	signal.signal(signal.SIGHUP, readConfiguration)
    signal.signal(signal.SIGINT, receiveSignal)
    signal.signal(signal.SIGQUIT, receiveSignal)
    signal.signal(signal.SIGILL, receiveSignal)
    signal.signal(signal.SIGTRAP, receiveSignal)
    signal.signal(signal.SIGABRT, receiveSignal)
    signal.signal(signal.SIGBUS, receiveSignal)
    signal.signal(signal.SIGFPE, receiveSignal)
    #signal.signal(signal.SIGKILL, receiveSignal)
    signal.signal(signal.SIGUSR1, receiveSignal)
    signal.signal(signal.SIGSEGV, receiveSignal)
    signal.signal(signal.SIGUSR2, receiveSignal)
    signal.signal(signal.SIGPIPE, receiveSignal)
    signal.signal(signal.SIGALRM, receiveSignal)
    signal.signal(signal.SIGTERM, terminateProcess)
```


