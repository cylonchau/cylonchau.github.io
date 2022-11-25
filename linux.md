# 操作系统类面试题收集


## Linux 基础


### 守护、僵⼫、孤⼉进程的概念

【答】

- 守护进程：运⾏在后台的⼀种特殊进程，独⽴于控制终端并周期性地执⾏某些任务。
- 僵⼫进程：⼀个进程 fork ⼦进程，⼦进程退出，⽽⽗进程没有 wait/waitpid⼦进程，那么⼦进程的进程描述符仍保存在系统中，这样的进程称为僵⼫进程。
- 孤⼉进程：⼀个⽗进程退出，⽽它的⼀个或多个⼦进程还在运⾏，这些⼦进程称为孤⼉进程。（孤⼉进程将由 init 进程收养并对它们完成状态收集⼯作）

### 进程间通讯方式有哪些？

- **Pipe**：无名管道，最基本的IPC，单向通信，仅在父/子进程之间，也就是将一个程序的输出直接交给另一个程序的输入。常见使用为 `ps -ef|grep xxx`
- **FIFO [`(First in, First out)`] 或  有名管道（`named pipe`）**:与Pipe不同，**FIFO**可以让两个不相关的进程可以使用FIFO。单向。
- **Socket 和 Unix Domain Socket**：socket和Unix套接字，双向。适用于网络通信，但也可以在本地使用。适用于不同的协议。
- **消息队列 `Message Queue`**:  SysV 消息队列、POSIX 消息队列。
- **Signal**: 信号，是发送到正在运行的进程通知以触发其事件的特定行为，是IPC的一种有限形式。
- **Semaphore**：信号量，通常用于IPC或同一进程内的线程间通信。他们之间使用队列进行消息传递、控制或内容的传递。（常见SysV 信号量、POSIX 信号量）
- **Shared memory**：（常见SysV 共享内存、POSIX 共享内存）。共享内存，是在进程（程序）之间传递数据的有效方式，目的是在其之间提供通信。


### BASH和DOS控制台之间的主要区别在于3个方面：

**答案：**

- BASH命令区分大小写，而DOS命令则不区分;
- 在BASH下，/ character是目录分隔符，\ 作为转义字符。在DOS下，/ 用作命令参数分隔符，\ 是目录分隔符
- DOS遵循命名文件中的约定，即8个字符的文件名后跟一个点，扩展名为3个字符。BASH没有遵循这样的惯例。
  
  

### Linux 中进程有哪几种状态？在 ps 显示出来的信息中，分别用什么符号表示的？

**答案：**

- R runnable (on run queue) 运行 (正在运行或在运行队列中等待)

- S 中断 sleeping(休眠中, 受阻, 在等待某个条件的形成或接受到信号)

- D 不可中断 uninterruptible sleep (usually IO)(收到信号不唤醒和不可运行, 进程必须等待直到有中断发生)

-  Z 僵尸 a defunct (”zombie”) process (进程已终止, 但进程描述符存在, 直到父进程调用wait4()系统调用后释放)

-  T 停止 traced or stopped (进程收到SIGSTOP, SIGSTP, SIGTIN, SIGTOU信号后停止运行运行) 

  

 

### 系统目前有许多正在运行的任务，在不重启机器的条件下，有什么方法可以把所有正在运行的进程移除呢？

答案：使用linux命令 ’disown -r ’可以将所有正在运行的进程移除。

### Linux 粘滞位作用

**答案：**

粘滞位(sticky bit)权限是针对目录的，对文件无效，设置了sticky位表示这个目录里的文件只能被owner和root删除。



### 什么是inode？

**答案：**

inode是Linux(Unix)操作系统中文件系统的一个概念。inode的全称为index node，也就是索引节点。那么inode是用来索引什么的呢？其实inode表示的是一个文件，它是用来索引文件数据的。



### 磁盘报错”No space left on device”，但是通过命令df –h查看磁盘空间没有满，请问为什么？

**答案：**

该磁盘的inode数量被用尽，无法再写入文件。
企业工作中邮件临时队列 `/var/spool/clientmquene`或`/var/spool/postfix/maildrop`这里很容易被大量小文件占满导致`No space left on device`的错误。`clientmquene`目录只有安装了`sendmail`服务，才会有，是`sendmail`的临时队列。



### 一个100M的磁盘分区，分别写入1K的文件，及写入1M的文件，分别可以写多少个？

**答案：**

在linux文件系统中，iNode用来存放文件的属性信息，而Block用来存放文件实际内容，默认大小1K(boot)或4K(非系统分区默认为4k)。

写入1M文件的数量为100/1，且不会存在磁盘浪费情况（这也说明了一般情况下，inode和block的数量都是足够的）；

而写入1K文件时，inode和block同时被消耗，但一般block数量远大于inode的数量，因此写入的数量就是inode的数量，并且这样会浪费3/4的磁盘容量。

### nohup

nohup 执行会忽略信号 `SIGHUP`，并将 `stdout/stderr` 重定向到文件 `nohup.out`。以便shell在关闭或注销后命令可以在后台继续运行 。nohup做的工作就是让 nohup 后的命令不在是当前 shell 的子命令。而是PPID=1的进程（进程的PPID=1）。这种情况下不能被带回到前台。

```c
signal (SIGHUP, SIG_IGN); // 忽略信号SIGHUP

char **cmd = argv + optind;
execvp (*cmd, cmd); // 在执行这个命令，而不是当前shell
```

对于输出的重定向，对于STDOUT/STDERR会忽略，然后写入到 `nohup.out`

```c
  ignoring_input = isatty (STDIN_FILENO);
  redirecting_stdout = isatty (STDOUT_FILENO);
  stdout_is_closed = (!redirecting_stdout && errno == EBADF);
  redirecting_stderr = isatty (STDERR_FILENO);

  /* If standard input is a tty, replace it with /dev/null if possible.
     Note that it is deliberately opened for *writing*,
     to ensure any read evokes an error.  */
  if (ignoring_input)
    {
      if (fd_reopen (STDIN_FILENO, "/dev/null", O_WRONLY, 0) < 0)
        error (exit_internal_failure, errno,
               _("failed to render standard input unusable"));
      if (!redirecting_stdout && !redirecting_stderr)
        error (0, 0, _("ignoring input"));
    }

  /* If standard output is a tty, redirect it (appending) to a file.
     First try nohup.out, then $HOME/nohup.out.  If standard error is
     a tty and standard output is closed, open nohup.out or
     $HOME/nohup.out without redirecting anything.  */
  if (redirecting_stdout || (redirecting_stderr && stdout_is_closed))
    {
      char *in_home = NULL;
      char const *file = "nohup.out";
      int flags = O_CREAT | O_WRONLY | O_APPEND;
      mode_t mode = S_IRUSR | S_IWUSR;
      mode_t umask_value = umask (~mode);
      out_fd = (redirecting_stdout
                ? fd_reopen (STDOUT_FILENO, file, flags, mode)
                : open (file, flags, mode));

```

> Referece [what is the function of the nohup command](https://askubuntu.com/questions/995179/what-is-the-function-of-the-nohup-command)


