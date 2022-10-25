# Linux VMware Tools详解 


## VMware Tools描述

VMware Tools 中包含一系列服务和模块，可在 VMware 产品中实现多种功能，从而使用户能够更好地管理客户机操作系统，以及与客户机系统进行无缝交互。

## 在Linux虚拟机中安装VMware Tools

**安装前准备**

- 虚拟机必须打开cd/dvd驱动器，否则`安装VMware Tools`的选项无法选择
- VMware Tools安装程序是使用Perl编写的，必须确认操作系统中安装Perl。

**安装步骤**

1. 在虚拟机菜单中右键单击虚拟机，然后单击客户机 > 安装/升级 VMware Tools。

2. 要创建一个挂载点 `mkdir /mnt/cdrom`

3. 要装载 CDROM，`mount /dev/cdrom /mnt/cdrom`

4. 要将安装文件文件复制到临时目录：`cp /mnt/cdrom/VMwareTools*.tar.gz /tmp/`；其中，`*` *部分是 VMware Tools 软件包的版本号，故以*替代*。

5. 解压文件：`cd /tmp && tar -zxvf VMwareTools*.tar.gz`

6. 运行PERL脚本以安装`VMware Tools`：`cd vmware-tools-distrib && ./vmware-install.pl`，*若要求选择，一路回车即可*。

7. 安装完成后清理 `rm -fr {/tmp/VMwareTools*,/tmp/vmware-tools-distrib} ; yum remove perl -y`，如不需要perl可以卸载

命令集合

```bash
mkdir /mnt/cdrom
mount /dev/cdrom /mnt/cdrom
cp /mnt/cdrom/VMwareTools*.tar.gz /tmp/
cd /tmp && tar -xf VMwareTools*.tar.gz
cd vmware-tools-distrib && ./vmware-install.pl
```

安装后清理

```bash
rm -fr {/tmp/VMwareTools*,/tmp/vmware-tools-distrib}
yum remove perl -y # 选择性执行
```

## VMware Tools服务

VMware Tools守护进程在后台运行。它在 Windows 客户机操作系统中名为 vmtoolsd.exe，在 Mac OS X 客户机操作系统中名为 vmware-tools-daemon，在 Linux、FreeBSD 和 Solaris 客户机操作系统中名为 vmtoolsd。

安装完成后，`VMware Tools`守护进程并未开机启动，可以设置开机启动，该守护进程在主机和客户机操作系统之间传递信息。

```
systemctl enable vmtoolsd
systemctl start vmtoolsd 
systemctl status vmtoolsd
```

## 配置虚拟机与宿主机系统之间的时间同步


启用时间同步时，VMware Tools会将虚拟机操作系统的时间设置为与宿主机的时间相同。

> 注意：无论 `VMware Tools`时间同步是否打开，在执行以下操作后都会进行时间同步：
>
> - 当启动 `VMware Tools`守护进程时，例如重新引导或打开电源操作过程中。
> - 在从某个挂起操作恢复虚拟机时
> - 恢复到快照后
> - 压缩磁盘后

**命令**

| 操作系统                  | 程序名称             |
| ------------------------- | -------------------- |
| Windows                   | VMwareToolboxCmd.exe |
| Linux、Solaris 和 FreeBSD | vmware-toolbox-cmd   |
| MAC OS X                  | vmware-tools-cli     |


```
vmware-toolbox-cmd timesync enable|disable
```

> **reference**
>
> [配置客户机与主机操作系统之间的时间同步](https://docs.vmware.com/cn/VMware-Tools/11.2.0/com.vmware.vsphere.vmwaretools.doc/GUID-C0D8326A-B6E7-4E61-8470-6C173FDDF656.html)
>
> [在 Linux 虚拟机中安装 VMware Tools (1018414)](https://kb.vmware.com/s/article/1018414?lang=zh_cn)
>
> [VMware Tools 产品文档](https://docs.vmware.com/cn/VMware-Tools/index.html)
