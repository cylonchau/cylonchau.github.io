# windows下Docker Desktop安装管理


##  检查要求

- Windows 10 企业版、专业版或教育版  （必须windows10 1903版本以上）版本号 `18362.1049&#43;` 或 `18363.1049&#43;` ，次版本＃大于.`1049`。最好是最新版（新版windows可以hype-v wsl2 vmvare共存，但安卓模拟器目前还没稳定的共存版本）。建议使用wsl2，安装包容量会比起hype-v小很多 。
- Windows开启wsl2，建议 Windows 10 2004（版本号不低于 19041.264），可wsl2与vmvare共存。
- CPU 支持并开启虚拟化（`Intel  VT-c` 或  `AMD SVM`）。
- 最少 4 GB 内存。

对于专业版、企业版、教育版可以使用docker desktop wsl2模式，此处无需开启`Hype-v`

![image-20210703212825867](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20210703212825867.png)

对于Win10 家庭版，Win10 19041.264之前版本，及 Win7 8用户，可以使用docker desktop `Hype-v` 后端。

##  修改安装盘

Docker Desktop 默认安装到 `C:\Program Files\Docker` 并不可更改，这样很不友好，可以通过软连接的方式改变Docker Desktop 默认安装盘。

```
mklink /J &#34;C:\Program Files\Docker&#34; &#34;D:\Program Files\Docker&#34;
```

## 限制wsl2运行最大内存

WSL 是 Microsoft 提供的一项功能，可以使开发人员能够直接在 Windows 上运行 `GNU/Linux` 环境，无需修改，无需传统虚拟机或双引导设置，减少了开发人员的使用复杂度

在 Docker Desktop 使用了 WSL 2 中的动态内存分配特性，极大地提高了资源消耗。这意味着，Docker Desktop 仅使用其所需的 CPU 和内存资源量，同时使 CPU 和内存密集型任务（例如构建容器）运行得更快。

但WSL2目前一个弊端，可能WSL2 vm会分配所有可用内存，并最终导致操作系统和其他应用程序的内存不足。

![img](https://miro.medium.com/max/700/1*GWlTOqcj4XeW4NO8_-OufA.png)

所以需要对WLS2内存和CPU资源进行限制，在 `cmd` 或 `powshell` 终端中

```
wsl --shutdown
notepad &#34;$env:USERPROFILE/.wslconfig&#34;
```

在用户目录创建一个文件`.wslconfig`  ，编辑 `.wslconfig` 

```
[wsl2]
memory=3GB   # 限制wsl2的虚拟机最大内存
processors=4  # 限制wsl2使用的处理器数量
swap=0      # 不使用交换文件
```

![img](https://miro.medium.com/max/700/1*Vv1RjqzQ5V7M2FzPQxdAkQ.png)

### 安装Docker Desktop

完成上面的操作，可以安装Docker Desktop了。从[Docker Desktop](https://www.docker.com/products/docker-desktop)网站下载安装Docker Desktop for Windows，大于500M。

安装步骤基本上点击操作即可，没有什么难度

## 镜像路径迁移

当使用了WSL2作为Docker Desktop后端引擎时，`WSL 2 Docker-Desktop-Data` 的VM磁盘镜像通常在 `%USERPROFILE%\AppData\Local\Docker\wsl\data\ext4.vhdx` 路径下，docker-desktop通常在`%LOCALAPPDATA%/Docker/wsl` 路径下，因为镜像的大小及一些交换文件，通常会占用大量C盘空间，可以改变其存储位置。

```
wsl --list -v
```

输入上述命令可以看到如下内容

```
  NAME                STATE          VERSION
* docker-desktop         Stopped         2
  docker-desktop-data      Stopped         2
```

`docker-desktop` 替换了之前使用的 Hyper-V VM 实现 Docker Desktop。这处理容器的引导和管理。

`docker-desktop-data` 是存储docker镜像和配置的地方；实际上是对 Hyper-V 以前使用的虚拟硬盘的直接替换。

从这里可以看出Docker Desktop使用了WSL2作为后端引擎时，实际上整个应用作为WLS2的两个子系统进行的。可以通过迁移WSL2系统镜像的存储位置来改变Docker霸占C盘不可转移的弊端。

导出wsl系统镜像

```
wsl --export docker-desktop docker-desktop.tar
wsl --export docker-desktop-data docker-desktop-data.tar
```

删除Docker Desktop wsl子系统，此操作会自动删除 `ext4.vhdx` 文件，故需要先导出一份备份

```
wsl --unregister docker-desktop
wsl --unregister docker-desktop-data
```

导入重新创建wsl Docker Desktop子系统

```
wsl --import docker-desktop d:\{new_path} docker-desktop.tar
wsl --import docker-desktop-data d:\{new_path} docker-desktop-data.tar
```

完成后，启动Docker服务，如果服务正常，可以删除掉 `docker-desktop.tar` 与 `docker-desktop-data.tar`

## 无法启动

我在使用windows时，会安装冰点还原，因为windows10 以上需要 冰点还原 8.38以上，我这里使用 8.38.020.4676 版本时，在开启还原状态时，Docker无法正常启动，在关闭还原时，可以正常启动。更换 8.62.020.5630。后正常。 8.38.020.4676 是2017年的版本，当时Docker对windows兼容并不好，而8.38.020.4676 是2020年发行的版本，目前在使用中并未发现异常。 8.38.020.4676 与 8.62.020.5630为网上常见的纯净的破解版了，所以按需选择使用。
