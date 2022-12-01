# 同步工具rsync使用示例




## Overview

官方网站：https://www.samba.org/ftp/rsync/rsync.html

rsync特性

- rsync类似于scp的功能

- rsync还可以在本地的不同分区和目录之间进行全量及增量的复制数据，类似cp又优于cp

- rsync可以实现文件的删除

一个rsync相当于 scp cp rm，但是还优于他们每一个

- 支持拷贝特殊文件如链接文件，设备等
- 可以有排除指定文件或目录的权限、时间、软硬链接、属主、组所有属性均不改变-p
- 可以有排除指定文件或目录的功能，相当于打包命令tar的排除功能
- 可以实现增量同步，即只同步发生变化的数据，因此数据传输效率很高tar。
- 可以使用rcp rsh ssh等方式来配合传输文件（rsync本身不对数据加密）
- 可以通过socket（进程方式）传输文件和数据
- 支持匿名的或认证（无须系统用户）的进程模式传输，可实现方便安全的进行数据备份及镜像

## 安装rsync

### linux上安装rsync

| Platform                                     | Command                    |
| -------------------------------------------- | -------------------------- |
| Debian/Ubuntu & Mint                         | sudo apt-get install rsync |
| Arch Linux                                   | pacman -S rsync            |
| Gentoo                                       | emerge sys-apps/rsync      |
| Fedora/CentOS/RHEL and Rocky Linux/AlmaLinux | sudo yum install rsync     |
| openSUSE                                     | sudo zypper install rsync  |

### windows安装rsync

官网下载cwRsync的服务端和客户端软件，cwRsync官网为：www.itefix.net/cwrsync

> Notes：由于伟大的people‘s leader president xi 网站已经无法中国地区访问（[点击测试](https://tcp.ping.pe/www.itefix.net:443)），伟大的俄罗斯因为俄乌战争，也不对俄罗斯访问了（俄乌战争开始后，西方大量学术网站禁止了俄罗斯地区的访问）

所以目前只能下载到一些镜像站上4.x版本，截止到2022年11月的6.2.7相差很多，windows客户端版本可以通过`chocolate` 安装

### rsync使用说明

rsync命令语法

| 选项   | 注释说明                           |
| ------ | ---------------------------------- |
| rsync  | rsync同步命令                      |
| option | 为同步时的选项参数                 |
| src    | 为源，及待拷贝的分区、文件或目录等 |
| dest   | 为目标分区、文件或目录等           |

rsync参数说明

| 参数选项                 | 注释说明                                                     |
| ------------------------ | ------------------------------------------------------------ |
| -v --verbose             | 详细模式输出，传输时的进度信息                               |
| -z --compress            | 传输时进行压缩以提高传输效率，`--compress-level=NUM` 可按级别压缩 |
| -a --archive             | 归档模式，表示以递归方式传输文件，并保持所有文件属性，等于-rtopgDl |
| r --recursive            | 对子目录进行递归模式，即目录下的所有目录都同样传输           |
| t --times                | 保持文件时间信息                                             |
| o --owner                | 保持文件属主信息                                             |
| p --perms                | 保持文件权限                                                 |
| g --group                | 保持文件属主信息                                             |
| P --progress             | 显示同步的过程及传输时的进度等信息                           |
| D --devices              | 爆出设备文件信息                                             |
| l --links                | 保留软连接                                                   |
| -e --rsh=**COMMAND**     | 使用的信道协议，制定替代rsh的shell程序，例如ssh              |
| -exclude=**PATTERN**     | 制定排除不需要传输的文件模式                                 |
| --bwlimit=**RATE**       | 限制socket I/O带宽                                           |
| --password-file=**PATH** | 指定密码文件，可避免多次输入密码                             |
| --delete                 | 如果服务器端为空，而客户端有文件，加上这个参数就会删除客户端目录下所有文件，相当于rm |
| --exclude                | 排除单/多个文件                                              |
| --timeout=**秒**         | 超时参数                                                     |
| --port                   | 指定端口                                                     |
| -R --relative            | 使用相对路径信息                                             |
| -u --update              | 仅仅进行更新，也就是跳过所有已经存在于DST，并且文件时间晚于要备份的文件。(不覆盖更新的文件) |

### rsync配置文件说明

```conf
# 每一个程序的进程都要依赖一个用户，默认为nobody，给默认为rsync
uid = rsync
gid = rsync
# 防止出现安全问题的，一般为局域网
use chroot = no
# 有多少个客户端可以连接服务器 同时连接的客户端
max connections = 200
# 客户端连接多少时间超时（如连接了不穿数据，多少时间踢掉）
timeout = 300
# linux中每个进程都对应一个进程号pid ，pid所在的文件就是pidfile，将来处理进程的时候不用再找了，直接杀pid文件就行
pid file = /var/run/rsyncd.pid
# rsync 在传输数据时，双方都在穿可能出错，在一个用户改的时候，其他用户不能改
lock file = /var/run/rsync.lock
# 日志，出错
log file = /var/log/rsyncd.log
# 模块 相当于nfs共享的目录 可以理解为nfs的
[oldboy]
path = /oldboy/
# 在传输过程中遇到错误自动忽略
ignore errors
# 可读可写 
read only = false
# 允不允许你列表 false不允许
list = false
# 允许的主机
hosts allow = 10.0.0.0/24
# 拒绝 rsync支持虚拟用户，不是系统用户，这
hosts deny = 0.0.0.0/32
# 传输时验证的用户
auth users = rsync_backup
# 用户对应的密码文件，如果指定密码文件，就得来回输入密码，在内网中不需要重复输入密码，就将密码写入文件
secrets file = /etc/rsync.password
```

## rsync的工作模式

rsync提供了三种同步模式：

- Rsync over SSH
- Rsync Daemon
- Local

## Local

rsync如果不使用远程模式类似于cp命令，例如

- `rsync /etc/hosts /opt/` ==  `cp /etc/hosts/ /opt/`

---

注：目录结尾加 `/` 与不加 `/` 的区别，不加斜线表示目录以及目录里面的东西，加斜线表示目录里面的东西

---

### rsync作为damon模式运行

如果主机没有运行 SSH服务，可以使用 `rsync --daemon` 配置并作为守护进程运行。这种场景下 `rsync` 监听端口 873 以获取来自其他使用 `rsync client` 的同步，这种模式下数据是未加密的。

#### 配置daemon端

**配置rsync daemon位置文件**

rsync damon模式配置文件默认在 `/etc/rsyncd.conf` 如果没有可以自行创建

```conf
lock file = /var/run/rsync.lock
log file = /var/log/rsyncd.log
pid file = /var/run/rsyncd.pid

[documents]
    path = /home/juan/Documents
    comment = The documents folder of Juan
    uid = juan
    gid = juan
    read only = no
    list = yes
    auth users = rsyncclient
    secrets file = /etc/rsyncd.secrets
    hosts allow = 192.168.1.0/255.255.255.0
```

rsyncd配置文件分为两部分，**全局参数**与**模块**。

- `lock file` 是 `rsync` 用于处理最大连接数的文件
- `log file` 是 `rsync` 同步时的日志；汇集路开始运行的时间, 其他客户端连接的时间与任何错误
- `pid file` 是 `rsync` 记录了进程ID，可以使用这个文件来kill进程

在全局参数之后，`[]` 中的是模块部分，代表的是要同步的一个目录，每个模块都是要共享的文件夹

- `[name]` 分配给模块的名称，每个模块代表一个目录，不能包含斜杠或右方括号
- `path` 同步的文件夹的路径
- `comment` 当客户端获取所有可用模块的时，获取出的列表内模块名称旁边的注释
- `uid` 当 rsync守护进程以root 身份运行时，可以指定以哪个用户拥有文件的权限
- `gid` 同上
- `read only` 决定rsync 客户端是否可以上传文件，默认所有模块为 true
- `list` 允许客户端请求可用模块列表，false 为从列表中隐藏模块。
- `auth users` 允许访问该模块内容的用户列表，用户以逗号分隔。用户不需要存在于系统中，他们由密钥文件定义
- `secrets file` 定义包含rsync同步时用户的用户名和密码的文件
- `hosts allow` 允许连接到rsync daemon的网段，默认允许所有主机连接

**创建rsync用户**

```bash
$ useradd rsync -s /sbin/nologin -M
```

**修改共享的目录的所属权限**

```bash
$ chown -R rsync.rsync /data  
```

**创建密码文件**

该文件中，每行代表一个rsync用户，账号和密码用冒号分隔

```bash
$ echo "rsync_backup:111111"> /etc/rsync.password
# 因密码可读，所以要降低权限
$ chmod 600 /etc/rsync.password
```

最后，更改此文件的权限，使其不能被其他用户读取或修改。如果此文件的权限设置不正确，`rsync` 会报错

```bash
sudo chmod 600 /etc/rsyncd.secrets
```

**启动 rsync daemon**

直接使用 `--daemon` 启动即可

```bash
sudo rsync --daemon
```

#### 使用rsync连接daemon一端

使用 `rsync` 命令连接 rsync daemon时，不可以像使用 SSH 时那样使用冒号，我们需要使用双冒号 ”::“，后跟模块名，以及同步的文件或文件夹，例如

```bash
rsync -rtv user@host::module/source/ destination/
```

另一种方法是增加 `rsync://` 协议，例如

```bash
rsync -rtv rsync://user@host/module/source/ destination/
```

如果不想输入密码可以指定参数 `--password-file=` 提供密码文件，密码文件就是创建的secret file，例如

```bash
$ rsync -avz rsync_backup@192.168.59.121::data \
	/test \
	--password-file=/etc/rsync.password 
```

使用rsync命令推与拉

- Pull `rsync [OPTION...] [USER@]HOST::SRC... [DEST]`

    ```bash
    rsync -avz rsync_backup@192.168.59.121::data /test/ --password-file=/etc/rsync.password
    ```

- Push `rsync [OPTION...] SRC... [USER@]HOST::DEST` || ` rsync [OPTION...] SRC... rsync://[USER@]HOST[:PORT]/DEST`

    ```bash
    rsync -avz /test/ rsync://rsync_backup@192.168.59.121/data/  --password-file=/etc/rsync.password
    ```

### Rsync Over SSH

除了daemon模式，也可以使用ssh模式进行传输，例如使用Over SSH命令

- Push  `rsync [OPTION]... -e ssh [SRC]... [USER@]HOST:DEST`
- Pull  `rsync [OPTION]... -e ssh [USER@]HOST:SRC... [DEST]`

较新版本的 rsync  SSH 为默认，可以省略该`-e ssh`选项，但 over ssh 与 over daemon的区别还是 一个冒号与两个冒号

## rsync使用实例

### Over SSH

- Pull    `rsync -avz -e 'ssh -p <port>' <user>@<host>:/<remote_dir> /<local_dir>`
- Push  `rsync -avz -e 'ssh -p <port>' /<local_dir> <user>@<host>:/<remote_dir> `

### Local to Remote

- Push  `rsync -avzh <local_dir> <system_user>@<host>:<remote_dir>`
- Pull    `rsync -avzh <system_user>@<host>:<remote_dir> <local_dir>`

### 显示进度条

- `rsync -avzhe ssh --progress <local_dir> <system_user>@<host>:<remote_dir>`

### Include and exclude

排除和包含可以使用文件，也可以使用正则表达式，例如

- 包含R开头文件： `rsync -avze  --include 'R*' <system_user>@<host>:<remote_dir> <local_dir>`
- 排除所有文件： `rsync -avze  --exclude '*' <system_user>@<host>:<remote_dir> <local_dir>`
- 仅上传R开头的文件：`rsync -avze ssh --include 'R*' --exclude '*' <system_user>@<host>:<remote_dir> <local_dir>`
- `--exclude-from` 是同步时排除文件中指定的文件

### 已存在文件的处理策略

如果想保证 `<Src>` 与 `<DST>`  保持一致可以使用 `--delete` 来删除 `<Src>` 现有文件或目录（只存在于目标目录不存在于源目标的文件）

```bash
rsync -avz --delete <system_user>@<host>:<remote_dir> <local_dir>
```

当在删除或更新目标目录已经存在的文件时，不想删除而想备份，可以指定参数 `-b`, `--backup`

### 限制传输文件大小

可以使用 “ **-–max-size** ” 选项指定要传输或同步的**最大文件大小**。例如限制最大为200k

```bash
rsync -avzhe ssh --max-size='200k' <local_dir> <system_user>@<host>:<remote_dir>
```

### 同步成功后自动删除源文件

删除发送方的文件：如果不想将备份的本地副本保留在服务器中，可以使用参数 “ **--remove-source-files** ” 来自动删除 `<src>` 的文件吗，一般用于Push场景中。

```bash
rsync --remove-source-files -zvh <local_dir> <system_user>@<host>:<remote_dir>
```

### dry run

与kubectl一样，rsync也支持 `--dry-run` ，该选项不会对文件进行更改但会显示命令的输出，可以与 `-v` 参数配合，这样就可以看到哪些内容会被同步

```bash
rsync --dry-run --remove-source-files -zvh <local_dir> <system_user>@<host>:<remote_dir>
```

### 限制传输时带宽

传输时，可以使用选项 “ **--bwlimit** ” 选项限制**I/O**带宽（默认单位KB），例如

```bash
rsync --bwlimit=100 -avzhe ssh <local_dir> <system_user>@<host>:<remote_dir>
```

### 忽略存在

如果目标目录中已经该文件，则忽略，例如

```bash
rsync --ignore-existing -zvh <local_dir> <system_user>@<host>:<remote_dir>
```

### 同步策略

默认rsync 只检查文件的大小和最后修改日期是否发生变化，如果发生变化，就重新传输；参数 `-c`，`--checksum` 可以改变`rsync` 的校验方式，会通过判断文件内容的校验和，决定是否重新传输。

参数 `--size-only` 可以使rsync只同步大小有变化的文件，不考虑文件修改时间的差异。

## troubleshooting

```
rsync error: some files/attrs were not transferred (see previous errors) (code 23) at main.c(1039) [sender=3.0.6]
```

问题原因：服务器端目录不存在。

解决：在服务器端创建目录即可解决

日志：`2017/05/07 08:02:16 [2852] rsync: chdir /data1 failed:No such file or directory (2)`

---

```
rsync: failed to set times on "." (in data1): Operation not permitted (1)
```

问题原因：服务器端模块目录权限不对

解决方法：将服务器模块目录权限更改为rsyncd.conf中的uid与gid

日志：`2017/05/07 08:07:49 [2862] rsync: mkstemp ".paichu.log.yOhm5e" (in data1) failed: Permission denied`

---

```
rsync: failed to connect to 192.168.59.121: No route to host (113)
rsync error: error in socket IO (code 10) at clientserver.c(124) [sender=3.0.6]
```

问题原因：防火墙处于开启状态

解决方法：关闭防火墙 `/etc/init.d/iptables stop`

---

```
@ERROR: auth failed on module data1

rsync error: error starting client-server protocol (code 5) at main.c(1503) [sender=3.0.6]
```

错误原因：

- 配置文件语法错误
- 密码与用户名错误
- 密码文件权限过大

解决方法：查看日志，查找具体问题

日志文件：`secrets file must not be other-acessible (see strict modes option)`

---

```
rsync: failed to connect to 192.168.59.121: Connection refused (111)
rsync error: error in socket IO (code 10) at clientserver.c(124) [receiver=3.0.6]
```

原因：rsync服务没开启

---

```
*** Skipping any contents from this failed directory ***

sent 485 bytes  received 14 bytes  332.67 bytes/sec
total size is 45994  speedup is 92.17
rsync error: some files/attrs were not transferred (see previous errors) (code 23) at main.c(1039) [sender=3.0.6]
```

原因：服务器同步目录权限不够或所属组

解决 chown rsync:rsync

---

原因：磁盘cache导致拷贝速度逐渐下降  <sup id="keywords"><a href="#1">[1]</a></sup>

rsync拷贝数据过程中发现一个现象：开始拷贝的时候速度很快，每秒有40MB左右，但拷贝几十分钟之后就降到10MB左右了，两边机器都没有跑什么应用，网络用netcat测也没有问题

然后我观察到的一个问题是两边的 `free` 命令都显示出内存占用很高，并且是`buffered/cache`一栏很高，因为这个缓存是可以手工释放的

```bash
sync; echo 3 > /proc/sys/vm/drop_caches
ssh root@new-repos 'sh -c "sync; echo 3 > /proc/sys/vm/drop_caches"'
```

补充说明

- linux操作系统会将文件系统的内容缓存起来，以便后面用到时加速，但在数据迁移场景下，基本上没有“后面用到时”这个场景，这个缓存反而碍事（TODO: 为什么导致网络io下降）
- 对于本机内大量拷贝文件，有人提供了一个 [nocache命令](https://github.com/Feh/nocache)  [Debian/Ubuntu ]( Debian/Ubuntu)已经收录了这个工具。它的功能是临时禁用cache，用法是将要执行的命令用 `nocache` 包住，比如: `nocache cp -a ~/ /mnt/backup/home-$(hostname)`，但rsync会使用网络通讯，所以`nocache rsync` 对远端没有作用（ *Note however, that rsync uses sockets, so if you try a nocache rsync, only the local process will be intercepted.）
- 也有人在多年以前给rsync提交了一个[补丁](https://bugzilla.samba.org/show_bug.cgi?id=9560 )，增加了 `--drop-cache` 选项，但遗憾的是没被接纳，说是过于 linux-specific，开发人员的意见（[见comment 3](https://bugzilla.samba.org/show_bug.cgi?id=9560#c3) ）是改用`nocache`: `nocache rsync -aiv --rsync-path='nocache rsync' some-host:/src/ /dest/`.  P.S. nocache工具的主页最后在[acknowledgements部分](https://github.com/Feh/nocache#acknowledgements) 说，其实它是衍生自rsync的这个补丁的。

---



> **Reference**
>
> <sup id="1">[1]</sup> [rsync用于数据迁移/备份的几个细节](https://www.cnblogs.com/bamanzi/p/rsync-gotchas.html)


