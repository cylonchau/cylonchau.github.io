# 适用于windows10 Linux子系统的安装管理配置




## 什么是WSL

`Windows Subsystem for Linux` 简称WLS，适用于Linux的Windows子系统，可以直接在Windows上运行Linux环境（包括大部分命令行工具）

### Linux containers与Windows Subsystem for Linux（WSL）区别

　　此处以docker与wsl进行一些比较，主要为个人的理解之处。

　　docker与wsl同样运行在本机环境中运行，不依赖其他管理程序与虚拟化。
　　docker与wsl同样为应用容器。

## 安装WSL

在Windows10上，用于Linux的Windows子系，可运行受支持的Linux版本（例如Ubuntu，OpenSuse，Debian等），而无需设置操作系统的复杂性。虚拟机或其他计算机。

### 使用设置为Linux启用Windows子系统

1. **打开设置**
2. **点击“应用”**。
3. **在“相关设置”部分下，单击“程序和功能”选项。**

![](https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/1380340-20201108233007245-1313186627.png)

4. **单击左窗格中的“打开或关闭Windows功能”选项。**

![](https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/1380340-20201108233019164-2093713154.png)

5. **检查Windows Subsystem for Linux选项。**

![](https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/1380340-20201108233050628-1621985294.png)

完成这些步骤后，将配置该环境以下载并运行Windows 10上的Linux版本。

### 使用Microsoft Store安装Linux发行版

要在Windows 10上安装Linux发行版，请使用以下步骤：

打开Microsoft Store。搜索要安装的Linux发行版。一些可用的发行版包括：

- [Ubuntu](https://www.windowscentral.com/e?link=https%3A%2F%2Fclick.linksynergy.com%2Fdeeplink%3Fid%3DkXQk6%252AivFEQ%26mid%3D24542%26u1%3DUUwpUdUnU72700YYw%26murl%3Dhttps%253A%252F%252Fwww.microsoft.com%252Fen-us%252Fp%252Fubuntu%252F9nblggh4msv6%26ourl%3Dhttps%253A%252F%252Fwww.microsoft.com%252Fstore%252FproductId%252F9NBLGGH4MSV6&token=ti9ZlPH4)
- [OpenSuse](https://www.windowscentral.com/e?link=https%3A%2F%2Fclick.linksynergy.com%2Fdeeplink%3Fid%3DkXQk6%252AivFEQ%26mid%3D24542%26u1%3DUUwpUdUnU72700YYw%26murl%3Dhttps%253A%252F%252Fwww.microsoft.com%252Fen-us%252Fp%252Fopensuse-leap-15-1%252F9njfzk00fgkv%26ourl%3Dhttps%253A%252F%252Fwww.microsoft.com%252Fstore%252FproductId%252F9NJFZK00FGKV&token=0vM0S7vS)
- [Kali Linux](https://www.windowscentral.com/e?link=https%3A%2F%2Fclick.linksynergy.com%2Fdeeplink%3Fid%3DkXQk6%252AivFEQ%26mid%3D24542%26u1%3DUUwpUdUnU72700YYw%26murl%3Dhttps%253A%252F%252Fwww.microsoft.com%252Fen-us%252Fp%252Fkali-linux%252F9pkr34tncv07%26ourl%3Dhttps%253A%252F%252Fwww.microsoft.com%252Fstore%252FproductId%252F9PKR34TNCV07&token=cG6n6K_6)
- [Debian](https://www.windowscentral.com/e?link=https%3A%2F%2Fclick.linksynergy.com%2Fdeeplink%3Fid%3DkXQk6%252AivFEQ%26mid%3D24542%26u1%3DUUwpUdUnU72700YYw%26murl%3Dhttps%253A%252F%252Fwww.microsoft.com%252Fen-us%252Fp%252Fdebian%252F9msvkqc78pk6%26ourl%3Dhttps%253A%252F%252Fwww.microsoft.com%252Fstore%252FproductId%252F9MSVKQC78PK6&token=BjqzImNu)
- [Alpine WSL](https://www.windowscentral.com/e?link=https%3A%2F%2Fclick.linksynergy.com%2Fdeeplink%3Fid%3DkXQk6%252AivFEQ%26mid%3D24542%26u1%3DUUwpUdUnU72700YYw%26murl%3Dhttps%253A%252F%252Fwww.microsoft.com%252Fen-us%252Fp%252Falpine-wsl%252F9p804crf0395%26ourl%3Dhttps%253A%252F%252Fwww.microsoft.com%252Fstore%252FproductId%252F9P804CRF0395&token=D9eFavB7)
- [Suse Linux Enterprise](https://www.windowscentral.com/e?link=https%3A%2F%2Fclick.linksynergy.com%2Fdeeplink%3Fid%3DkXQk6%252AivFEQ%26mid%3D24542%26u1%3DUUwpUdUnU72700YYw%26murl%3Dhttps%253A%252F%252Fwww.microsoft.com%252Fen-us%252Fp%252Fsuse-linux-enterprise-server-12%252F9p32mwbh6cns%26ourl%3Dhttps%253A%252F%252Fwww.microsoft.com%252Fstore%252FproductId%252F9P32MWBH6CNS&token=J65kn0Rn)

2. 选择要在您的设备上安装的Linux发行版。
3. 单击获取（或安装）按钮。
4. Microsoft Store安装Linux发行版
5. 单击启动按钮。为Linux发行版创建一个用户名，然后按Enter键。
6. 指定发行版的密码，然后按Enter。
7. 重复密码，然后按Enter确认。

完成以上步骤后，即完成安装了WLS（没有图形界面），在开始菜单 运行 `wls` 启动。

### 离线安装WLS

　　官网指导手册内包含所支持的[Linux离线安装包](https://docs.microsoft.com/en-us/windows/wsl/install-manual#downloading-distros)

> 这里下载的为`Ubuntu 18.04`，下载后，文件格式为appx格式，本次使用的操作系统为，windows1709企业版，并且卸载了所有的 UWP应用。因此只能使用命令行进行安装。
>
> 非LTSC企业版或卸载windows store的可以直接双击安装

　　管理员打开Powershell 运行以下命令，将路径替换为下载的离线安装包路径。本次安装的wls默认安装到C盘

```powershell
Add-AppxPackage .\app_name.appx
```
查看已经安装的子系统
```
wslconfig /l
```

#### 安装时选择其他盘安装


1. 首先解压.appx文件

2. 用 [LxRunOffline](https://github.com/DDoSolitary/LxRunOffline/releases) 安装：

windows10 1803以上版本下载最新版即可，windows 1709及一下，可以安装[2.x](https://github.com/DDoSolitary/LxRunOffline/releases?after=v2.2.2)版本。

3. 使用以下命令安装，`-f`后的文件为解压后文件内根目录的`install.tar.gz`

语法
```
LxRunOffline.exe install  -n <install systemname> -d <save path> -f <unzip_path/install.tar.gz>
```

```
LxRunOffline.exe install -n ubuntu -d d:\wls -f d:\Ubuntu_1804.2019.522.0_x64\install.tar.gz
```

等运行完成后（warning可忽略），开始 => 运行`wls`进入，进入后默认就是root用户。另外开始菜单不会有单独的启动的图标。

#### 如何在重装系统后恢复原来的WSL

```
.\LxRunOffline.exe rg -n ubuntu -d D:\wsl\ubuntu
```

## 配置wsl与windows共用开发环境

　　本次配置的开发环境为golang与goland，在windows下与linux下的环境开发与运行为相同的环境。其他的开发环境类似。

　　因为wsl共享windows的路径，可以再windows与linux安装golang编译器。并分别设置`go env`

### windows

```
set GO111MODULE=on
set GOPATH=D:\go_work
set GOPROXY=https://goproxy.io,https://goproxy.cn,direct
set GOROOT=C:\Go
```
Linux，GOPATH要与windows设置为同一个路径，这样可以保证安装的包为同一个。即实现了同一个开发环境与Linux环境。
```
export GO111MODULE=on
export GOPROXY=https://goproxy.io,https://goproxy.cn,direct
export GOROOT=/usr/local/go
export GOPATH=/mnt/d/go_work/
export PATH=$PATH:$GOROOT/bin:$GOPATH/bin
```

### goland设置

file => setting => Tools => Terminal 
```
C:\Windows\System32\wsl.exe
```
![](https://img2020.cnblogs.com/blog/1380340/202011/1380340-20201109000212690-34842593.png)

file => setting => Editor => Code Style
![](https://img2020.cnblogs.com/blog/1380340/202011/1380340-20201109000303092-1063825125.png)

### goland wls terminal .bashrc不生效 

　　在wsl中发现一些环境变量、shell颜色等都不生效。这里需要了解shell的类型

　　shell有两种类型，`Login Shell`和`Non Login Shell`。每一个shell都有自己自定义的脚本来预设值shell运行的环境。

#### Login Shell

　　当成功登陆用户后，将创建登陆shell（通过ssh sudo 或者 terminal）

　　查看当前shell是什么类型的shell `echo $0`

- Login Shell：-bash或-su。
- Non Login Shell： bash或su

**Login shell 登陆后执行以下脚本：**

> 登陆执行`/etc/profile`
> /etc/profile执行`/etc/profile.d`中的所有脚本
> 然后执行用户 `~/.bash_profile`
> `~/.bash_profile` 会有命令执行用户目录 `~/.bashrc`
> `~/.bashrc`中会执行 `/etc/bashrc`

#### Non Login Shell

　　`Non Login Shell`是由`Login Shell`启动的shell。例如，登陆成功后执行bash，此时是`Non Login Shell`

**Non Login Shell登陆后执行以下脚本：**

> 首先执行 `~/.bashrc`
> 然后 `~/.bashrc` 执行 `/etc/bashrc`
> `/etc/bashrc` 调用 `/etc/profile.d` 中的脚本

　　了解了执行顺序后，按照步骤查看对应问题所在，此处问题没有`~/.bashrc`中设置的alias和颜色。根据`Login shell`流程应为`~/.bash_profile`中去执行`~/.bashrc`，查看`~/.bash_profile` 发现文件为空。

　　复制一份linux `~/.bash_profile` 中的文件内容到对应的`~/.bash_profile`后发现功能已经正常实现。

```bash
# .bash_profile

# Get the aliases and functions
if [ -f ~/.bashrc ]; then
	. ~/.bashrc
fi

# User specific environment and startup programs

PATH=$PATH:$HOME/bin
```

![](https://img2020.cnblogs.com/blog/1380340/202012/1380340-20201214220914329-1063628897.png)
