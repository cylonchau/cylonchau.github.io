# WSL与Windows环境共享


在使用wsl时，总是需要执行windows的cmd，但是windows命令行对于大多数人使用起来还是不习惯，微软提供了在windows中Linux与Windows的命令互通，即可以使用cmd shell执行Linux命令，也可以使用bash shell来执行windows命令。

WSL可对 Windows 与 Linux 之间的集成操作：
- 从 Linux shell（如 Ubuntu）运行 Windows 工具（任意 `.exe`）。
- 从 Windows shell（即 PowerShell or cmd ）运行 Linux 命令（如 cd ls grep）。
- 在 WSL与windows之间共享环境变量。 （版本 17063&#43;）

满足上述要求，可以很好地使用windows的软件在WSL中畅快的操作，即空WSL环境拥有了python解析器 docker等操作。

## 如何在 WSL和 Windows 之间共享环境变量

从`Build 17063` 开始，可以利用 `WSLENV` 来增强 Win/WSL 之间的环境变量互操作。

### 什么是WSLENV

- WSLENV 是一个以冒号分隔的环境变量列表，当从 WSL 启动 WSL进程或 Win进程时包含的变量
- 每个变量都可以以斜杠作为后缀，后跟标识位以指定它的转换方式
- WSLENV 可以在 WSL 和 Win32 之间转换的路径
- WSLENV。在WSL中，是以冒号分隔的列表。在Win中，是以分号分隔的列表
- 可以在`.bashrc`或者windows自定义环境变量中设置`WSLENV`

例如：一个`WSLENV`应该设置为
```
WSLENV=GOPATH/l:USERPROFILE/w:SOMEVAR/wp
```

在17063之前，WSL访问Windows环境变量唯一方法是使用全路径（可以使用全路径从WSL下启动Win32可执行文件）。但是没有办法在WSL中设置环境变量，调用Win进程，并期望将该变量传送到进程。

在17063之后，引入一个名为`WSLENV`的特殊环境变量，以帮助WSL和Win之间的共享。 `WSLENV`存在于两个环境中。用户可以将WSLENV的值设置为耦合值与环境变量串联，每个都以 `\` 为标志，以指定应该如何解析该变量。例如：

## /p

`/p` 表示应在WSL和Win32之间转换path。例如。在WSL中设置变量，将其添加到WSLENV设置`/p` 标志，然后在win环境cmd.exe中读取变量，该值会随着rootfs的转变而转换为对应的值。


```bash
$ /mnt/d# export TRANSLATABLE=`pwd`
$ /mnt/d# echo $TRANSLATABLE
/mnt/d
$ /mnt/d# export WSLENV=TRANSLATABLE\p
$ /mnt/d# export WSLENV=TRANSLATABLE/p
$ /mnt/d# echo $WSLENV
TRANSLATABLE/p
$ /mnt/d# cmd.exe
Microsoft Windows [版本 10.0.19043.1052]
(c) Microsoft Corporation。保留所有权利。

D:\&gt;set TRANSLATABLE # 在windows中查看环境变量
TRANSLATABLE=D:\
```

### /l

`/l` 表示该值是路径列表（如Linux的PATH）。在Linux中，是以冒号分隔的路径列表。在Win中，是以分号分隔的路径列表。`/l` 可以将路径列表适当对不通系统进行转换。

```bash
$ /mnt/d# export TEMPORARY=/usr/local/go/bin:/usr/local/python/bin

$ /mnt/d# WSLENV=$WSLENV:TEMPORARY/l

$ /mnt/d# echo $WSLENV
TRANSLATABLE/p:TEMPORARY/l

$ /mnt/d# cmd.exe
Microsoft Windows [版本 10.0.19043.1052]
(c) Microsoft Corporation。保留所有权利。

D:\&gt;set TEMPORARY
TEMPORARY=\\wsl$\ubuntu1\usr\local\go\bin;\\wsl$\ubuntu1\usr\local\python\bin

```
### /u

`/u` 表示仅在Linux（WSL）中调用变量的值为 Win 类型的变量值，及windows向Linux传递环境变量，但格式不变

```bash
D:\compose&gt;set zhangsan=D:\compose

D:\compose&gt;set zhangsan
zhangsan=D:\compose

D:\compose&gt;set WSLENV=zhangsan/u

D:\compose&gt;wsl -d ubuntu1
$ /mnt/d/compose# echo $zhangsan
D:\compose
```

&gt; 如需要自动适应转换，则需要 使用`/up`

### /w

`/w` 表示仅在从Win调用WSL环境变量是的值，该参数并不会自动转换，如需转换一样需要使用 `/wp` 。


```bash
$ /mnt/d/compose# export FROMWSL=/mnt/d/compose
$ /mnt/d/compose# export WSLENV=FROMWSL/w
$ /mnt/d/compose# cmd.exe
Microsoft Windows [版本 10.0.19043.1052]
(c) Microsoft Corporation。保留所有权利。

D:\compose&gt;set FROMWSL
FROMWSL=/mnt/d/compose

D:\compose&gt;exit
$ /mnt/d/compose# export WSLENV=FROMWSL/wp
$ /mnt/d/compose# cmd.exe
Microsoft Windows [版本 10.0.19043.1052]
(c) Microsoft Corporation。保留所有权利。

D:\compose&gt;set FROMWSL
FROMWSL=D:\compose
```

## 使用脚本传递变量

如果需要BASH脚本传递对应的变量到windows程序执行，例如

```bash
#!/bin/bash

export MYPATH=/mnt/c/Users/

WSLENV=$WSLENV:MYPATH/p cmd.exe /c set MYPATH
```

通过WSL shell环境执行，可以得到windows程序处理的结果，并且可以拿到环境变量

```bash
$ /mnt/d/compose# bash 1.sh
MYPATH=C:\Users\
```

## 实例：设置一个开发环境，使其共享环境变量

例如，希望在WSL中设置DEV环境。使用WSLENV VAR，将其配置为在WSL和Win之间共享GoPath。

### 安装golang

首先，我们需要安装两个平台。要在Windows与WSL安装，步骤不说了。（如果是python等解析语言，可以使用alias直接使用windows的解析器则不需要安装了）

### 设置项目

接下来，需要配置的GO项目。该项目需要在Windows文件系统下。在PowerShell中发出以下命令：(这里在桌面配置的)

```bash
mkdir $env:USERPROFILE\desktop\goProject
cd $env:USERPROFILE\desktop\goProject
New-Item hello.go
```

配置环境变量，然后将gopath添加到WSLENV，此时，两个文件系统间，会使用同一个GOPATH

```bash
setx GOPATH &#34;$env:USERPROFILE\desktop\goProject&#34;
setx WSLENV &#34;$env:WSLENV:GOPATH&#34;/p
```

&gt; 需要事项
&gt; - WSL（通过.profile或其他）中的定义将在通过WSL访问时覆盖默认WSLENV中定义的值。
&gt; - 在关闭WSL后，WSLENV不会持久化，需要修改相应的配置文件（.profile，.bash_rc等）。
&gt; - WSL可以设置任何值。如果仅设置当前文件系统变量，则不会自动转换。通过WSLENV可以自动翻译成两种不通的文件系统下的环境变量。

## 题外话cmd.exe 跨文件系统常用参数

| options | describe                                      |
| ------- | --------------------------------------------- |
| /C      | 使用cmd.exe运行一个命令并终止，类似于 bash -c |

&gt; **Reference**
&gt; 更多cmd.exe帮助参考 [cmd_helps](https://ss64.com/nt/cmd.html)
&gt; [WSL备份及windows Docker安装](https://www.cnblogs.com/woki/p/14967604.html)
&gt; [WSL安装维护](https://www.cnblogs.com/woki/p/13946476.html)


