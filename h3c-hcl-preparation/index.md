# H3C Cloud与WSL2共存

[toc]

## Background

由于Windows10 开启WSL2后无法和 *eNSP* 做到兼容，但是 *H3C HCL* 在版本 **HCL_v2.1.2.1** 提供了 **VirtualBox 6.0.14** 作为虚拟化后端，理论上来说可以做到 WSL2 与 HCL 共存。

并且开启了WSL2后并于其他虚拟化平台（VirtualBox, Vmvare）做到兼容的情况下，这种情况大部分禁止套娃（虚拟化下在虚拟化），通过安装虚拟机的方式再安装 *eNSP* 发现启动不报错，但是很长时间起不来。

&gt; Notes &lt;sup&gt;&lt;a href=&#34;#1&#34;&gt;[1]&lt;/a&gt;&lt;/sup&gt;：HCL官方给的建议是，对于windows7装的版本为*HCL_v2.1.1*；对于Windows10 并且开启了 *Hype-v* 或者 *Dokcer-Desktop*，推荐使用 *HCL_v3.0.1.1*

下载地址：[HCL Download](http://www.h3c.com/cn/Service/Document_Software/Software_Download/Other_Product/H3C_Cloud_Lab/Catalog/HCL/)

## 安装过程

下载好安装时，直接下一步直至完成即可，VirtualBox已被内嵌至安装包内了。

![image-20220811201205362](../../../../images/ch1%20environment%20preparation/image-20220811201205362.png)

&lt;center&gt;图：HCL安装界面&lt;/center&gt;

&gt; Notes：如果需要抓包，自行安装*Wireshark*，安装好后，在*HCL*设置中配置 `wireshark.exe` 的路径即可

## VirtualBox启用hyper-v支持 &lt;sup&gt;&lt;a href=&#34;#2&#34;&gt;[2]&lt;/a&gt;&lt;/sup&gt;

进入VirtualBox安装目录, 确定当前目录下存在VBoxManage.exe文件, 在当前目录打开powershell. 或者你将VBoxManage.exe所在目录加入环境变量, 任意路径下打开powershell.

```powershell
# 或指定vbox所有虚拟系统开启
./VBoxManage.exe setextradata global &#34;VBoxInternal/NEM/UseRing0Runloop&#34; 0
```

开启后，HCL所有的设备就工作正常了，这种情况下也不用牺牲*WSL2*或者*Dokcer-Desktop*。因为eNSP官方没有再更新，导致hype-v与VirtualBox无法兼容，暂时无解。

&gt; **Reference**
&gt;
&gt; &lt;sup id=&#34;1&#34;&gt;[1]&lt;/sup&gt; [H3C Cloud Lab](https://www.h3c.com/en/d_201811/1129464_294551_0.htm)
&gt;
&gt; &lt;sup id=&#34;2&#34;&gt;[2]&lt;/sup&gt; [Windows 10 (2004) 启用wsl2, 并与VirtualBox 6.0&#43;共存](https://blog.csdn.net/qq_36992069/article/details/104750248)






