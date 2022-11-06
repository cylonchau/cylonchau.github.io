# systemd

[TOC]

## 一、关于Linux服务管理
Linux系统从启动到提供服务的过程是这样，先是机器加电，然后通过MBR或者UEFI加载GRUB，再启动内核，内核启动服务，然后开始对外服务。
SysV init UpStart systemd主要是解决服务引导管理的问题。
* CentOS 5：SysV init
* CentOS 6：Upstart
* CentOS 7：Systemd
http://www.linuxidc.com/Linux/2015-04/115937.htm

### 1.1 SysV init的优缺点
SysV init是最早的解决方案，依靠划分不同的运行级别，启动不同的服务集，服务依靠脚本控制，并且是顺序执行的。

SysV init方案的优点：
* 1.原理简单，易于理解；
* 2.依靠shell脚本控制，编写服务脚本门槛比较低。

缺点是：
* 1.服务顺序启动，启动过程比较慢。
* 2.不能做到根据需要来启动服务，比如通常希望插入U盘的时候，再启动USB控制的服务，这样可以更好的节省系统资源。

### 1.2 UpStart的改进
为了解决系统服务的即插即用，UpStart应运而生，在CentOS6系统中，SysV init和UpStart是并存的，UpStart主要解决了服务的即插即用。服务顺序启动慢的问题，UpStart的解决办法是把相关的服务分组，组内的服务是顺序启动，组之间是并行启动。

### 1.3 systemd的诞生

SysV init服务启动慢，在以前并不是一个问题，尤其是Linux系统以前主要是在服务器系统上，常年也难得重启一次。有的服务器光硬件检测都需要5分钟以上，相对来说系统启动已经很快了。

但是随着移动互联网的到来，SysV init服务启动慢的问题显得越来越突出，许多移动设备都是基于Linux内核，比如安卓。移动设备启动比较频繁，每次启动都要等待服务顺序启动，显然难以接受，systemd就是为了解决这个问题诞生的。

**systemd的设计思路是：**

* 尽可能的快速启动服务。
* 尽可能的减少系统资源占用。

### 1.4 为什么systemd能做到启动很快

systemd使用并行的方法启动服务，不像SysV init是顺序执行的，所以大大节省了系统启动时间。

使用并行启动，最大的难点是要解决服务之间的依赖性，systemd的解决办法是使用类似缓冲池的办法。比如对TCP有依赖的服务，在启动的时候会检查依赖服务的TCP端口，systemd会把对TCP端口的请求先缓存起来，当依赖的服务器启动之后，在将请求传递给服务，使两个服务通讯。同样的进程间通讯的D-BUS也是这样的原理，目录挂载则是先让服务以为目录被挂载了，到真正访问目录的时候，才去真正操作。

## 二、systemd的特性

**systemd解决了那些问题？**

* 按需启动服务，减少系统资源消耗；
* 尽可能并行启动进程，减少系统启动等待时间；
* 提供一个一致的配置环境，不光是服务配置；
* 提供服务状态快照，可以恢复特定点的服务状态。

## 三、CentOS 7的systemd特性

### 3.1 套接字服务保持激活功能
在系统启动的时候，systemd为所有支持套接字激活功能的服务创建监听端口，当服务启动后，就将套接字传给这些服务。这种方式不仅可以允许服务在启动的时候平行启动，也可以保证在服务重启期间，试图连接服务的请求，不会丢失。对服务端口的请求被保留，并且存放到队列中。
### 3.2 进程间通讯保持激活功能
当有客户端应用第一次通过D-Bus方式请求进程间通讯时，systemd会立即启动对应的服务。systemd依据D-Bus的配置文件使用进程间通讯保持激活功能。
### 3.3 设备保持激活功能
当特定的硬件插入时，systemd启动对应的硬件服务支持。systemd依据硬件服务单元配置文件保持硬件随时被激活。
### 3.4 文件路径保持激活功能
当特定的文件或者路径状态发生改变的时候，systemd会激活对应的服务。systemd依据路径服务单元配置文件保证服务被激活。
### 3.5 系统状态快照
systemd可以临时保存当前所有的单元配置文件，或者从前一个快照中恢复单元配置文件。为了保存当前系统服务状态，systemd可以动态的生成单元文件快照。
### 3.6 挂载和自动挂载点管理
systemd监控和管理挂载和自动挂载点，并根据挂载点的单元配置文件进行挂载。
### 3.7 闪电并行启动
因为使用套接字保持激活功能，systemd可以并行的启动所以套接字监听服务，大大减少系统启动时间。
### 3.8 单元逻辑模拟检查
当激活或者关闭一个单元，systemd会计算依赖行，产生一个临时的模拟检查，并且校验一直性。如果不一致，systemd会尝试自动修正，并且移除报错的不重要的任务。
### 3.9 和SysV init向后兼容
systemd完全支持SysV init Linux标准的基础核心规范脚本，这样的脚本易于升级到systemd服务单元。


## 四、核心概念:unit
### 4.1 什么是单元
在RHEL7之前，服务管理是分布式的被SysV init或UpStart通过 &lt;font color=&#34;#f8070d&#34; size=3&gt;`/etc/rc.d/init.d`&lt;/font&gt; 下的脚本管理。这些脚本是经典的Bash脚本，允许管理员控制服务的状态。在RHEL7中，这些脚本被服务单元文件替换。

在systemd中，服务、挂载等资源统一被称为单元，所以systemd中有许多单元类型，服务单元文件的扩展名是.service，同脚本的功能相似。例如有查看、启动、停止、重启、启用或者禁止服务的参数。

配置文件进行标识和配置：文件中主要包含了系统服务、监听socket、保存的系统快照及其他与init相关的信息。

**systemd单元文件放置位置：**
```sh
/usr/lib/systemd/system/systemd		# 默认单元文件安装目录
/run/systemd/system					      # 单元运行时创建，这个目录优先于安装目录
/etc/systemd/system					      # 系统管理员创建和管理的单元目录，优先级最高。
```

### 4.2 Unit类型



| 类型           | 详解-                                                 |
| -------------- | ----------------------------------------------------- |
| Service unit   | 文件扩展名为service，用于定义系统服务。               |
| Target unit    | 文件扩展名为.target，用于模拟实现“运行级别”           |
| Device unit    | 文件扩展名为.device，用于定义内核识别的设备。         |
| Mount unit     | 文件扩展名为.mount，定义文件系统挂载点                |
| Socket unit    | 文件扩展名为.socket，用于表示进程间通信用的socket文件 |
| Snapshot unit  | 文件扩展名为.sanpshot，管理系统快照                   |
| Swap unit      | 文件扩展名为.swap，用于表示swap设备                   |
| Automount unit | 文件扩展名为.automount，文件系统的自动挂载点          |
| Path unit      | 文件扩展名为.path，用于定义文件系统中的一个文件或目录 |

`.service` 与服务对应的后缀名为service的unit、文件，无需执行权限，仅仅为systemd的配置文件。当systemd探测到有进程访问时，按需激活这个服务的机制，任何依赖与这个服务的其他服务想启动的话，

服务的并行启动：

`.device` 在某个硬件设备被激活或变得可用时，从而激活服务

`.path`：某个文件路径变得可用或里面有文件时（文件发生变动）激活某个服务

**系统快照：**

systemd能将所有unit当前状态保存到临时文件中（临时保存到一个持久设备上）。启动时，可从保存的快照开始继续向后运行。 必要时能自动载入。

**向后兼容：**

sysV init脚本。（能够兼容 start、stop restart status至少这4个服务脚本）以前启动服务的脚本放到centos7里直接可以用。

## 五、CentOS 7的systemd向后兼容

systemd被设计成尽可能向后兼容SysV init和Upstart，下面是一些特别要注意的和之前主要版本的RHEL不再兼容的部分。

### 5.1 systemd对运行级别支持有限
为了保存兼容，systemd提供有限target单元，“模拟”一些运行级别，也可以被早期的分布式的运行级别命令支持。不是所有的target都可以被映射到运行级别，在这种情况下，使用runlevel命令有可能会返回一个为N的不知道的运行级别，所以推荐尽量避免在RHEL7中使用runlevel命令。

### 5.2 systemd不支持像init脚本那样的个性化命令。
除了一些标准命令参数例如：start、stop、status，SysV init脚本可以根据需要支持想要的任何参数，通过参数提供附加的功能，因为SysV init的服务器脚本实际上就是shell脚本，命令参数实际上就是shell子函数。

举个例子，RHEL6的iptables服务脚本可以执行panic命令行参数，这个参数可以让系统立即进入紧急模式，丢弃所有的进入和发出的数据包。但是类似这样的命令行参数在systemd中是不支持的，systemd只支持在配置文件中指定命令行参数。

### 5.3 systemd不支持和没有从systemd启动的服务通讯。
当systemd启动服务的时候，他保存进程的主ID以便于追踪，systemctl工具使用进程PID查询和管理服务。相反的，如果用户从命令行启动特定的服务，systemctl命令是没有办法判断这个服务的状态是启动还是运行的。
### 5.4 systemd可以只停止运行的服务
在RHEL6及之前的版本，当关闭系统的程序启动之后，RHEL6的系统会执行/etc/rc0.d/下所有服务脚本的关闭操作，不管服务是处于运行或者根本没有运行的状态。而systemd可以做到只关闭在运行的服务，这样可以大大节省关机的时间。

### 5.5 不能从标准输出设备读到系统服务信息。
systemd启动服务的时候，将标准输出信息定向到/dev/null，以免打扰用户。
### 5.6 systemd不继承任何上下文环境。
systemd不继承任何上下文环境，如用户或者会话的HOME或者PATH的环境变量。每个服务得到的是干净的上下文环境。
### 5.7 SysV init脚本依赖性
当systemd启动SysV init脚本，systemd在运行的时候，从LinuxStandardBase(LSB)Linux标准库头文件读取服务的依赖信息并继承。
### 5.8 超时机制
为了防止系统被卡住，所有的服务有5分钟的超时机制。


## 六、systemd服务管理
使用systemcl命令可以控制服务，service命令和chkconfig命令依然可以使用，但是主要是出于兼容的原因，应该尽量避免使用service命令和chkconfig命令。

使用systemctl命令的时候，服务名字的扩展名可以写全，例如：
```sh
systemctl stop httpd.service
```
也可以忽略，例如：
```sh
systemctl stop httpd
```

### 6.1 常用命令

### 6.2 服务管理

| 说明                       | 命令                                                         |
| -------------------------- | ------------------------------------------------------------ |
| 启动服务                   | service name start ==	systemctl start name.service        |
| 停止服务                   | service name stop	== systemctl stop name.service          |
| 重启服务                   | service name restart	== systemctl restart name.service    |
| 查看服务状态               | service name status == systemctl status name.service         |
| 条件式重启                 | service name condrestart == systemctl try-restart name.service |
| 重载或重启服务             | systemctl reload-or-restart name.service                     |
| 重载或条件式重启           | systemctl reload-or-try-restart name.service                 |
| 禁止设定为开机自启动       | systemctl mask name.service                                  |
| 取消设定为开机自启动       | systemctl unmask name.service                                |
| 查看服务当前激活与否的状态 | systemctl is-active name.service                             |
| 允许服务开机启动           | systemctl enable name.service                                |
| 禁止服务开机启动           | systemclt disable name.service                               |
| 级别切换                   | systemctl list-units --type target                           |
| 获取默认运行级别           | systemctl get-default                                        |
| 修改默认级别               | systemctl set-default name.service                           |
| 切换至紧急救援模式         | systemctl rescue                                             |
| 切换至emergency模式        | systemctl emergency                                          |
| 关机                       | systemctl halt systemctl poweroff                            |
| 重启                       | systemctl reboot                                             |
| 挂起                       | systemctl suspend                                            |
| 快照                       | systemctl hibernate                                          |
| 快照并挂起                 | systemctl hybrid-sleep                                       |



### 6.3 查看服务详细信息


查看所有已激活的服务。默认只列出处于激活状态的服务，如果希望看到所有的服务，使用--all或-a参数：
```sh
systemctl list-units --type service
```
查看所有的服务
```sh
systemctl list-units --type service --all
```
检查服务开机启动状态
```sh
systemctl status name.service
systemctl is-enabled name.service
```
列出所有服务并且检查是否开机启动
```sh
systemctl list-unit-files --type service
```
选项重新加载所以单元文件并重新创建依赖书，在需要立即应用单元文件改变的时候使用。另外，也可以使用init q的命令达到同样的目的。还有，如果修改的是一个正在运行服务的单元文件，服务需要被重启下：
```sh
systemct lrestart name.service
systemctl daemon-reload
```
查看服务依赖关系
```sh
systemctl list-dependencies
```

## 七、systemd target

在RHEL7之前的版本，使用运行级别代表特定的操作模式。运行级别被定义为七个级别，用数字0到6表示，每个级别可以启动特定的一些服务。RHEL7使用target替换运行基本。

systemd target使用target单元文件描述，target单位文件扩展名是.target，target单元文件的唯一目标是将其他systemd单元文件通过一连串的依赖关系组织在一起。举个例子，graphical.target单元，用于启动一个图形会话，systemd会启动像GNOME显示管理(gdm.service)、帐号服务（axxounts-daemon）这样的服务，并且会激活multi-user.target单元。相似的multi-user.target单元，会启动必不可少NetworkManager.service、dbus.service服务，并激活basic.target单元。

RHEL7预定义了一些target和之前的运行级别或多或少有些不同。为了兼容，systemd也提供一些target映射为SysV init的运行级别，具体的对应信息如下：



| 代码 | 命令                               | 说明                         |
| ---- | ---------------------------------- | ---------------------------- |
| 0    | runlevel0.target,poweroff.targe    | 关闭系统。                   |
| 1    | runlevel1.target,rescue.target     | 进入救援模式。               |
| 2    | runlevel2.target,multi-user.target | 进入非图形界面的多用户方式。 |
| 3    | runlevel3.target,multi-user.target | 进入非图形界面的多用户方式。 |
| 4    | runlevel4.target,multi-user.target | 进入非图形界面的多用户方式。 |
| 5    | runlevel5.target,graphical.target  | 进入图形界面的多用户方式。   |
| 6    | runlevel6.target,reboot.target     | 重启系统。                   |
|      |                                    |                              |

***
注：对于systemd来说 234没有区别
***

### 7.1 target管理

使用如下命令查看目前可用的target：
```sh
systemctl list-units --type target
```
改变当前的运行基本使用如下命令：
```sh
systemctl isolate name.target
systemctl isolate rescue.target
[lc@lnmp ~]$ runlevel 
1 3
```

### 7.2 修改默认的运行级别

使用systemctl get-default命令得到默认的运行级别：
```sh
[lc@lnmp ~]$ systemctl get-default 
multi-user.target
```
使用systemctl set-default name.target修改默认的运行级别：
```sh
systemctl set-default graphical.target 
# 可以看到。默认级别就是操作如下两步骤
rm &#39;/etc/systemd/system/default.target&#39;
ln-s&#39;/usr/lib/systemd/system/graphical.target&#39;&#39;/etc/systemd/system/default.target&#39;
```
使用 Target 的时候，systemctl list-dependencies命令和systemctl isolate命令也很有用。
查看 multi-user.target 包含的所有服务
```sh
systemctl list-dependencies multi-user.target
```
一般来说，常用的 Target 有两个：一个是multi-user.target，表示多用户命令行状态；另一个是graphical.target，表示图形用户状态，它依赖于multi-user.target。官方文档有一张非常清晰的 Target 依赖关系图。

### 7.3 Target 的配置文件
Target 也有自己的配置文件。
```sh
[Unit]
Description=Multi-User System
Documentation=man:systemd.special(7)
Requires=basic.target		# Requires字段：要求basic.target一起运行。

# 冲突字段。如果rescue.service或rescue.target正在运行，multi-user.target就不能运行，反之亦然。
Conflicts=rescue.service rescue.target

# 表示multi-user.target在basic.target 、 rescue.service、 rescue.target之后启动，如果它们有启动的话。
After=basic.target rescue.service rescue.target

# 允许使用systemctl isolate命令切换到multi-user.target。
AllowIsolate=yes
```
注意，Target 配置文件里面没有启动命令。

### 7.4 救援模式和紧急模式
使用进入救援模式，如果连救援模式都进入不了，可以进入紧急模式：
```sh
systemctl rescue 	  # 救援模式（进入救援模式，如果连救援模式都进入不了，可以进入紧急模式）
systtmctl emergency # 紧急模式
```
紧急模式进入最小的系统环境，以便于修复系统。紧急模式根目录以只读方式挂载，不激活网络，只启动很少的服务，进入紧急模式需要root密码。


## 八、关闭、暂停、休眠系统

RHEL7中，使用systemctl替换一些列的电源管理命令，原有的命令依旧可以使用，但是建议尽量不用使用。systemctl和这些命令的对应关系为：



| 说明                   | SysV/Upstart      | system                 |
| ---------------------- | ----------------- | ---------------------- |
| 停止系统（关机）       | halt              | systemctl hatl         |
| 关闭系统，关闭系统电源 | poweroff          | systemctl poweroff     |
| 重启系统               | reboot            | systemctl reboot       |
| 暂停系统               | pm-suspend        | systemctl suspend      |
| 休眠系统               | pm-hibernate      | systemct lhibernate    |
| 暂停并休眠系统         | pm-suspend-hybrid | systemctl hybrid-sleep |

## 九、通过systemd管理远程系统

不光是可以管理本地系统，systemd还可以控制远程系统，管理远程系统主要是通过SSH协议，只有确认可以连接远程系统的SSH，在systemctl命令后面添加-H或者--host参数，加上远程系统的ip或者主机名就可以。

## 十、创建和修改systemd单元文件

### 10.1 单元文件概述
一个服务怎么启动，完全由它的配置文件决定。下面就来看，配置文件有些什么内容。

配置文件主要放在/usr/lib/systemd/system目录，也可能在/etc/systemd/system目录。找到配置文件以后，使用文本编辑器打开即可。

下面以sshd.service文件为例，它的作用是启动一个 SSH 服务器，供其他用户以 SSH 方式登录。

```sh
[Unit]
Description=OpenSSH server daemon
Documentation=man:sshd(8) man:sshd_config(5)
After=network.target sshd-keygen.service
Wants=sshd-keygen.service

[Service]
EnvironmentFile=/etc/sysconfig/sshd
ExecStart=/usr/sbin/sshd -D $OPTIONS
ExecReload=/bin/kill -HUP $MAINPID
Type=simple
KillMode=process
Restart=on-failure
RestartSec=42s

[Install]
WantedBy=multi-user.target
```

```sh
[root@lnmp etc]# systemctl status rsyncd
rsyncd.service - fast remote file copy program daemon
   Loaded: loaded (/usr/lib/systemd/system/rsyncd.service; disabled) 
   # loaded 服务已经被加载，显示单元文件绝对路径，标志单元文件可用。
   # disabled表示开机不允许启动Status服务的附件信息。
   Active: active (running) since 四 2017-01-26 21:59:41 CST; 5min ago
   # active表示当前状态  从什么时间被激活
 Main PID: 1360 (rsync) # main pid 与进程名字一致的PID，主进程PID。进程可能有多个进程可能有一组
   CGroup: /system.slice/rsyncd.service
           └─1360 /usr/bin/rsync --daemon --no-detach
# cgroup表示资源组，启动命令是/usr/bin/rsync --daemon --no-detach
1月 26 21:59:41 lnmp systemd[1]: Starting fast remote file copy program daemon...
1月 26 21:59:41 lnmp systemd[1]: Started fast remote file copy program daemon.

[lc@lnmp ~]$ systemctl status rsyncd
rsyncd.service - fast remote file copy program daemon
   Loaded: loaded (/usr/lib/systemd/system/rsyncd.service; disabled)
   Active: inactive (dead)/
```

上面的输出结果含义如下:&#34;&#34;
* Loaded行：配置文件的位置，是否设为开机启动
* Active行：表示正在运行
* Main PID行：主进程ID
* Status行：由应用本身（这里是 httpd ）提供的软件当前状态
* Cgroup块：应用的所有子进程
* 日志块：应用的日志

### 10.2 理解单元文件结构

### 10.3 单元文件概述
可以看到，配置文件分成几个区块，每个区块包含若干条键值对。
典型的单元文件包含三节：

* **[Unit]**：包含不依赖单元类型的一般选项，这些选型提供单元描述，知道单元行为，配置单元和其他单元的依赖性。
* **[unittype]**：如果单元有特定的类型指令，在unittype节这些指令被组织在一起。举个例子，服务单元文件包含[Service]节，里面有经常使用的服务配置。

* **[Install]**：包含systemctl enable或者disable的命令安装信息。

#### 10.3.1 [Unit]节选项



| 字段          | 说明                                                         |
| ------------- | ------------------------------------------------------------ |
| Description   | 字段给出当前服务的简单描述                                   |
| Documentation | 给出文档位置。                                               |
|               | 启动顺序 **&lt;font color=&#34;#0215cd&#34; size=2&gt; 注意:After和Before字段只涉及启动顺序，不涉及依赖关系&lt;/font&gt;** |
| After         | 表示sshd.service应该在network.target sshd-keygen.service之后启动。 |
| Before        | 定义sshd.service应该在哪些服务之前启动。                     |
|               | 举例来说，某 Web 应用需要 postgresql 数据库储存数据。在配置文件中，它只定义要在 postgresql 之后启动，而没有定义依赖 postgresql 。上线后，由于某种原因，postgresql 需要重新启动，在停止服务期间，该 Web 应用就会无法建立数据库连接。 |
|               | **&lt;font color=&#34;#0215cd&#34; size=2&gt; 设置依赖关系，需要使用Wants字段和Requires字段。&lt;/font&gt;** |
| Wants         | 表示sshd.service与sshd-keygen.service之间存在&lt;font style=&#34;background:#ffff00;&#34; size=2&gt; &#34;弱依赖&#34; &lt;/font&gt;关系，即如果&#34;sshd-keygen.service&#34;启动失败或停止运行，不影响sshd.service继续执行。 |
| Requires      | 表示&lt;font style=&#34;background:#ffff00;&#34; size=2&gt; &#34;强依赖&#34; &lt;/font&gt;关系，即如果该服务启动失败或异常退出，那么sshd.service也必须退出。 |
|               | **&lt;font color=&#34;#0215cd&#34; size=2&gt;注意，Wants字段与Requires字段只涉及依赖关系，与启动顺序无关，默认情况下是同时启动的。&lt;/font&gt;** |

#### 10.3.2 [Service] 区块：启动行为

&amp;nbsp;Service区块定义如何启动当前服务。

&gt; **启动命令**

&amp;nbsp;许多软件都有自己的环境参数文件，该文件可以用EnvironmentFile字段读取。

      EnvironmentFile字段：指定当前服务的环境参数文件。该文件内部的key=value键值对，可以用$key的形式，在当前配置文件中获取。

&amp;nbsp;上面的例子中，sshd 的环境参数文件是&lt;font color=&#34;#f8070d&#34; size=3&gt;`/etc/sysconfig/sshd`&lt;/font&gt;。

&amp;nbsp;配置文件里面最重要的字段是ExecStart。

      ExecStart：定义启动进程时执行的命令。

上面的例子中，启动sshd，执行的命令是&lt;font color=&#34;#f8070d&#34; size=3&gt;`/usr/sbin/sshd -D $OPTIONS`&lt;/font&gt;，其中的变量$OPTIONS就来自EnvironmentFile字段指定的环境参数文件。

与之作用相似的，还有如下这些字段：

| 选项            | 说明                                                         |
| --------------- | ------------------------------------------------------------ |
| ExecStart       | 指定启动单元的命令或者脚本，ExecStartPre和ExecStartPost节指定在ExecStart之前或者之后用户自定义执行的脚本。Type=oneshot允许指定多个希望顺序执行的用户自定义命令。 |
| ExecStop        | 指定单元停止时执行的命令或者脚本。                           |
| ExecReload      | 指定单元重新加载是执行的命令或者脚本。                       |
| ExecStartPre    | 启动服务之前执行的命令                                       |
| ExecStartPost   | 启动服务之后执行的命令                                       |
| ExecStopPost    | 停止服务之后执行的命令                                       |
| Restart         | 这个选项如果被允许，服务重启的时候进程会退出，会通过systemctl命令执行清除并重启的操作。 |
| RemainAfterExit | 如果设置这个选择为真，服务会被认为是在激活状态，即使所以的进程已经退出，默认的值为假，这个选项只有在Type=oneshot时需要被配置。 |

请看下面的例子。
```sh
[Service]
ExecStart=/bin/echo execstart1
ExecStart=
ExecStart=/bin/echo execstart2
ExecStartPost=/bin/echo 1
ExecStartPost=/bin/echo 2
```
上面这个配置文件，第二行ExecStart设为空值，等于取消了第一行的设置，运行结果如下。

```sh
[root@lnmp ~]# systemctl start test 
[root@lnmp ~]# systemctl status test
test.service - test daemon
   Loaded: loaded (/usr/lib/systemd/system/test.service; disabled)
   Active: inactive (dead)

1月 30 07:11:16 lnmp systemd[1]: Starting test daemon...
1月 30 07:11:16 lnmp systemd[1]: Started test daemon.
1月 30 07:11:16 lnmp echo[1822]: test1_start
1月 30 07:11:16 lnmp echo[1824]: test1_stop
```

所有的启动设置之前，都可以加上一个连词号（-），表示&lt;font style=&#34;background:#ffff00;&#34; size=2&gt;&#34;抑制错误&#34;&lt;/font&gt;，即发生错误的时候，不影响其他命令的执行。比如，&lt;font color=&#34;#f8070d&#34; size=3&gt;`EnvironmentFile=-/etc/sysconfig/sshd`&lt;/font&gt;（注意等号后面的那个连词号），就表示即使/etc/sysconfig/sshd 文件不存在，也不会抛出错误。

&gt; **启动类型**

Type字段定义启动类型。它可以设置的值如下。

| 选项值  | 说明                                                         |
| ------- | ------------------------------------------------------------ |
| simple  | 默认值，ExecStart字段启动的进程为主进程                      |
| forking | 进程作为服务主进程的一个子进程启动，父进程在完全启动之后退出。 |
| oneshot | 类似于simple，但只执行一次，进程在启动单元之后随之退出。     |
| dbus    | 类似于simple，但随着单元启动后只有主进程得到D-BUS名字。      |
| notify  | 类似于simple，启动结束后会发出通知信号，然后 Systemd 再启动其他服务 |
| idle    | 类似于simple，但是要等到其他任务都执行完，才会启动该服务。一种使用场合是为让该服务的输出，不与其他服务的输出相混合 |
|         |                                                              |

下面是一个oneshot的例子，笔记本电脑启动时，要把触摸板关掉，配置文件可以这样写。
```sh
[Unit]
Description=Switch-off Touchpad

[Service]
Type=oneshot
ExecStart=/usr/bin/touchpad-off

[Install]
WantedBy=multi-user.target
```

上面的配置文件，启动类型设为oneshot，就表明这个服务只要运行一次就够了，不需要长期运行。

如果关闭以后，将来某个时候还想打开，配置文件修改如下。

```sh
[Unit]
Description=Switch-off Touchpad

[Service]
Type=oneshot
ExecStart=/usr/bin/touchpad-off start
ExecStop=/usr/bin/touchpad-off stop
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```

上面配置文件中，RemainAfterExit字段设为yes，表示进程退出以后，服务仍然保持执行。这样的话，一旦使用systemctl stop命令停止服务，ExecStop指定的命令就会执行，从而重新开启触摸板。

&gt; **重启行为**

&amp;nbsp;Service区块有一些字段，定义了重启行为。
```sh
  KillMode字段：定义 Systemd 如何停止 sshd 服务。
```

上面这个例子中，将KillMode设为process，表示只停止主进程，不停止任何sshd 子进程，即子进程打开的 SSH session 仍然保持连接。这个设置不太常见，但对sshd很重要，否则你停止服务的时候，会连自己打开的SSH session一起杀掉。

KillMode字段可以设置的值如下：

| 选项值        | 说明                                                 |
| ------------- | ---------------------------------------------------- |
| control-group | （默认值）	当前控制组里面的所有子进程，都会被杀掉 |
| process       | 只杀主进程                                           |
| mixed         | 主进程将收到 SIGTERM 信号，子进程收到 SIGKILL 信号   |
| none          | 没有进程会被杀掉，只是执行服务的 stop 命令。         |

&gt; **Restart字段。**

&amp;nbsp;Restart字段：定义了 sshd 退出后，Systemd 的重启方式。

上面的例子中，Restart设为on-failure，表示任何意外的失败，就将重启sshd。如果 sshd 正常停止（比如执行systemctl stop命令），它就不会重启。

Restart字段可以设置的值如下。

| 选项值      | 说明                                                         |
| ----------- | ------------------------------------------------------------ |
| no          | 默认值；退出后不会重启                                       |
| on-success  | 只有正常退出时（退出状态码为0），才会重启                    |
| on-failure  | 非正常退出时（退出状态码非0），包括被信号终止和超时，才会重启 |
| on-abnormal | 只有被信号终止和超时，才会重启                               |
| on-abort    | 只有在收到没有捕捉到的信号终止时，才会重启                   |
| on-watchdog | 超时退出，才会重启                                           |
| always      | 不管是什么退出原因，总是重启                                 |

&amp;nbsp;对于守护进程，推荐设为on-failure。对于那些允许发生错误退出的服务，可以设为on-abnormal。

&gt; **RestartSec字段。**

&amp;nbsp;RestartSec字段：表示 Systemd 重启服务之前，需要等待的秒数。上面的例子设为等待42秒。

&lt;h5 id=&#34;10.2.3&#34;&gt;[Install] 区块
&amp;nbsp;说明：Install区块，定义如何安装这个配置文件，即怎样做到开机启动。
WantedBy字段：表示该服务所在的 Target。

Target的含义是服务组，表示一组服务。WantedBy=multi-user.target指的是，sshd 所在的 Target 是multi-user.target。

这个设置非常重要，因为执行systemctl enable sshd.service命令时，sshd.service的一个符号链接，就会放在/etc/systemd/system目录下面的multi-user.target.wants子目录之中。

Systemd 有默认的启动 Target。

```sh
[root@lnmp ~]# systemctl get-default
multi-user.target
```

上面的结果表示，默认的启动 Target 是multi-user.target。在这个组里的所有服务，都将开机启动。这就是为什么systemctl enable命令能设置开机启动的原因。

&gt; **修改配置文件后重启**

修改配置文件以后，需要重新加载配置文件，然后重新启动相关服务。

```sh
# 重新加载配置文件
systemctl daemon-reload
```


### 10.4 一个postfix服务的例子：

单元文件位于&lt;font color=&#34;#f8070d&#34; size=3&gt;`/usr/lib/systemd/system/postifix.service`&lt;/font&gt;，内容如下：

```sh
[Unit] 
Description=PostfixMailTransportAgent 
After=syslog.targetnetwork.target 
Conflicts=sendmail.serviceexim.service 

[Service] 
Type=forking 
PIDFile=/var/spool/postfix/pid/master.pid 
EnvironmentFile=-/etc/sysconfig/network
ExecStartPre=-/usr/libexec/postfix/aliasesdb
ExecStartPre=-/usr/libexec/postfix/chroot-update
ExecStart=/usr/sbin/postfixstart
ExecReload=/usr/sbin/postfixreload
ExecStop=/usr/sbin/postfixstop

[Install] 
WantedBy=multi-user.target
```

#### 10.5 创建自定义的单元文件

以下几种场景需要自定义单元文件：
* 希望自己创建守护进程；
* 为现有的服务创建第二个实例；
* 引入SysV init脚本。

另外一方面，有时候需要修改已有的单元文件。下面介绍创建单元文件的步骤：

&gt; 1. 准备自定义服务的执行文件。

可执行文件可以是脚本，也可以是软件提供者的的程序，如果需要，为自定义服务的主进程准备一个PID文件，一保证PID保持不变。另外还可能需要的配置环境变量的脚本，确保所以脚本都有可执行属性并且不需要交互。

&gt; 2.在&lt;font color=&#34;#f8070d&#34; size=3&gt;`/etc/systemd/system/`&lt;/font&gt;目录创建单元文件，并且保证只能被root用户编辑

```sh
touch /etc/systemd/system/mariadb.service
chmod 644 /etc/systemd/system/mariadb.service
```

***
**&lt;font color=&#34;#f8070d&#34; size=3&gt;注：文件不需要执行权限&lt;/font&gt;。**
***

&gt; 3. 打开name.service文件，添加服务配置，各种变量如何配置视所添加的服务类型而定，下面是一个依赖网络服务的配置例子：

```sh
[Unit] 
Description=mariadb multi demo 3306
After=network.target

[Service] 
ExecStart=/data/3306/mysql start
ExecReload=/data/3306/mysql restart
ExecStop=/data/3306/mysql stop
Type=forking
PIDFile=/data/3306/mysqld.pid

[Install] 
WantedBy=multi-user.target
```

&gt; 4.通知systemd有个新服务添加：

```sh
systemctl daemon-reload 
systemctl start name.service
```

&lt;h4 id=&#34;10.5&#34;&gt;10.5 创建第二个sshd服务的例子

&gt; **1.拷贝sshd_config文件**

```sh
cp /etc/ssh/sshd{,-second}_config
# {,second} 等于 和second ，类似与 {a,c}的用法
```

&gt; **2.编辑sshd-second_config文件，添加22220的端口，和PID文件：**

```
Port 22220 
PidFile /var/run/sshd-second.pid
```

如果还需要修改其他参数，请阅读帮助。

&gt; **3.拷贝单元文件：**

```sh
 cp /usr/lib/systemd/system/sshd{,-second}.service
```

&gt; **4.编辑单元文件sshd-second.service**

```sh
[Unit] 
Description=OpenSSH server second instance daemon 
After=syslog.target network.targe tauditd.service sshd.service 

[Service] 
EnvironmentFile=/etc/sysconfig/sshd
ExecStart=/usr/sbin/sshd -D -f /etc/ssh/sshd-second_config $OPTIONS 
ExecReload=/bin/kill -HUP $MAINPID 
KillMode=process 
Restart=on-failure 
RestartSec=42s 

[Install] 
WantedBy=multi-user.target
```

&gt; **5.如果使用SELinux，添加tcp端口，负责第二sshd服务的端口就会被拒绝绑定：**

```sh
semanage port -a -tssh_port_t -p tcp22220
```

&gt; &gt; **6.设置开机启动并测试：**

```sh
systemctl enable sshd-second.service 
ssh -p 22220 user@server
```

#### 10.6 修改已经存在的单元文件

systemd unit配置文件默认保存在/usr/lib/systemd/system/目录，不建议直接修改这个目录下的文件，自定义的文件在/etc/systemd/system/目录下，如果有扩展的需求，可以使用以下方案：

创建一个目录/etc/systemd/system/unit.d/，这个是最推荐的一种方式，可以参考初始的单元文件，通过附件配置文件来扩展默认的配置，对默认单元文件的升级会被自动升级和应用。

从/usr/lib/systemd/system/拷贝一份原始配置文件到/etc/systemd/system/，然后修改。复制的版本会覆盖原始配置，这种方式不能增加附件的配置包，用于不需要附加功能的场景。

如果需要恢复到默认的配置文件，只需要删除/etc/systemd/system/下的配置文件就可以了，不需要重启机器。




## 参考资料

- http://www.linuxidc.com/Linux/2015-04/115937.htm
- http://www.jinbuguo.com/systemd/index.html
- http://www.ruanyifeng.com/blog/2016/03/systemd-tutorial-part-two.html
