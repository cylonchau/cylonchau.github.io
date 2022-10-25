# 如何使用golang通过进程ID找到进程名称

> 一个很好的问题：How golang to get process name by process id (pid)? 

目前看来go api并没有提供通过pid获取进程名称的方法，可以通过 `/proc/<pid>/cmdline`来获取对应的进程名称，也可以通过 `readlink /proc/6530/exe` 来获取

- `/proc/<pid>/cmdline` 获取的为运行进程的名称，通常包含一些特殊字符。例如 `"-bash\x00"`，`sshd: root@pts/0`
- `readlink /proc/6530/exe` 获取的为对应进程运行的程序的路径

```go
pid := os.Getppid()
contents, err := ioutil.ReadFile(fmt.Sprintf("/proc/%d/cmdline",pid))
```

```go
pid := os.Getppid()
contents, err := os.Readlink(fmt.Sprintf("/proc/%d/cmdline",pid))
```

> Reference
> [process name from pid](https://superuser.com/questions/632979/if-i-know-the-pid-number-of-a-process-how-can-i-get-its-name)
