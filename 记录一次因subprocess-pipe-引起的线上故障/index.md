# 由PIPE size 引起的线上故障


&gt; **sence**：python中使用subprocess.Popen(cmd, stdout=sys.STDOUT, stderr=sys.STDERR, shell=True) ，stdout, stderr 为None.

在错误中执行是无法捕获 stderr的内容，后面将上面的改为 `subprocess.Popen(cmd, stdout=PIPE, stderr=PIPE, shell=True)`,发现是可以拿到 `stderr`, 但是会遇到大量任务hanging，造成线上事故。

为此特意查询`subprocess`的一些参数的说明。

&gt; `stdin` `stdout ` `stderr ` 如果这些参数为 `PIPE`, 此时会为一个文件句柄，而传入其他（例如 `sys.stdout` 、`None` 等）的则为`None`

正如这里介绍的一样，[subprocess](https://docs.python.org/2/library/subprocess.html#subprocess.Popen.stdin) 。

而使用 `PIPE`，却导致程序 hanging。一般来说不推荐使用 `stdout=PIPE`  `stderr=PIPE`，这样会导致一个死锁，子进程会将输入的内容输入到 `pipe`，直到操作系统从buffer中读取出输入的内容。

查询手册可以看到确实是这个问题 [Refernce](https://docs.python.org/2/library/subprocess.html#subprocess.Popen.communicate)

&gt; **Warning** This will deadlock when using `stdout=PIPE` and/or `stderr=PIPE` and the child process generates enough output to a pipe such that it blocks waiting for the OS pipe buffer to accept more data. Use [`communicate()`](https://docs.python.org/2/library/subprocess.html#subprocess.Popen.communicate) to avoid that.

而在linux中 `PIPE` 的容量（capacity）是内核中具有固定大小的一块缓冲区，如果用来接收但不消费就会阻塞，所以当用来接收命令的输出基本上100% 阻塞所以会导致整个任务 hanging。*（ -Linux2.6.11 ，pipe capacity 和system page size 一样（如， i386 为 4096 bytes ）。 since Linux 2.6.11&#43;，pipe capacity 为 65536  bytes。）*

关于更多的信息可以参考：[pipe](https://linux.die.net/man/7/pipe)

所以如果既要拿到对应的输出进行格式化，又要防止程序hang，可以自己创建一个缓冲区，这样可以根据需求控制其容量，可以有效的避免hanging。列如：

```python
cmd = &#34;this is complex command&#34;
outPipe = tempfile.SpooledTemporaryFile(bufsize=10*10000)
fileno = outPipe.fileno()
process = subprocess.Popen(cmd,stdout=fileno,stderr=fileno,shell=True)
```

另外，几个参数设置的不通的区别如下：

`stdout=None` 为继承父进程的句柄，通俗来说为标准输出。

`stderr=STDOUT` 重定向错误输出到标准输出

`stdout=PIPE` 将标准输出到linux pipe

&gt; **Reference**
&gt;
&gt; [subprocess](https://docs.python.org/2/library/subprocess.html#subprocess.Popen.stdin)
&gt;
&gt; [subprocess stderr/stdout field is None](https://stackoverflow.com/questions/25370347/python-subprocess-stderr-stdout-field-is-none-if-created)
&gt;
&gt; [subprocess-popen-hanging](https://stackoverflow.com/questions/39477003/python-subprocess-popen-hanging)
&gt;
&gt; [pipe size](https://unix.stackexchange.com/questions/11946/how-big-is-the-pipe-buffer)
