# debian install tutorial


## Preparation

debian11几乎可以使用任何旧的计算机硬件，因为最小安装的要求非常低。以下是最低要求和推荐要求：

| 最低要求                                                     | 推荐要求                                                     |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 存储：10 Gigabytes&lt;br&gt;内存：512 Megabytes&lt;br&gt;CPU: 1 GigaHertz | 存储：10 Gigabytes&lt;br/&gt;内存：2 Gigabytes&lt;br/&gt;CPU: 1 GigaHertz or more |

### 如何选择下载安装包

- [offical mirror](https://www.debian.org/CD/http-ftp/#stable)
- [aliyun mirror](https://developer.aliyun.com/mirror/debian)

官网提供了安装包的下载，其中CD是网络安装，DVD是离线安装

![image-20220802174122790](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20220802174122790.png)

&lt;center class=&#34;podsc&#34;&gt;debian官方下载页面&lt;/center&gt;

&gt; Notes：CD安装包很小，下载下来是 `debian-11.4.0-amd64-netinst.iso` 如名所示，这是一个网络安装包，所以推荐下载DVD部分，可以达到离线安装的效果

## 安装步骤

在界面中选择“Install”，安装将开始。如果图形化安装可以选择“Graphical install”，这里选择“Install”。

![image-20220802124054641](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20220802124054641.png)

&lt;center class=&#34;podsc&#34;&gt;欢迎页面&lt;/center&gt;



完成后，系统将提示选择安装时的“语言”。选择喜欢的语言，然后按“Enter”。这里选择英文

![image-20220802124137343](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20220802124137343.png)

&lt;center class=&#34;podsc&#34;&gt;选择语言页面&lt;/center&gt;

这将是接下来安装步骤

![image-20220802124701368](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20220802124701368.png)

&lt;center class=&#34;podsc&#34;&gt;安装步骤概述&lt;/center&gt;

### 选择位置与键盘布局

选择区域

![image-20220802124200648](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20220802124200648.png)

&lt;center class=&#34;podsc&#34;&gt;选择区域&lt;/center&gt;

下面部署时选择键盘布局：中国大陆使用的键盘布局是美国-英语，不要选择英国-英语之类，布局是不一样的，会存在按键输出的结果会不同

![image-20220802124219600](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20220802124219600.png)

&lt;center class=&#34;podsc&#34;&gt;选择键盘布局&lt;/center&gt;

完成上述操作后，将开始加载镜像。等待扫描完成。。。。

![image-20220802124450675](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20220802124450675.png)

&lt;center class=&#34;podsc&#34;&gt;等待扫描组件&lt;/center&gt;

### 设置主机名和域名

这步骤中将配置一个“主机名”。与一个“域”名称。![image-20220802124523526](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20220802124523526.png)

&lt;center class=&#34;podsc&#34;&gt;配置主机名&lt;/center&gt;

“域” 可以选择留空确定

![image-20220802124552521](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20220802124552521.png)

&lt;center class=&#34;podsc&#34;&gt;配置域&lt;/center&gt;

完成上述操作后，安装程序将提示需要设置 root 密码。输入您的 root 密码，然后在重新输入以进行验证后继续。

![image-20220802124732048](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20220802124732048.png)

&lt;center class=&#34;podsc&#34;&gt;设置Root密码&lt;/center&gt;

### 设置非ROOT用户名、账户和密码

下一步创建一个非ROOT用户，这个步骤是必须的，并为这个新创建的帐户分配一个密码。以下截图将描述将如何完成此操作。 

![image-20220802124814435](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20220802124814435.png)

&lt;center class=&#34;podsc&#34;&gt;配置普通用户&lt;/center&gt;

为这个用户配置密码

![image-20220802124841150](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20220802124841150.png)

&lt;center class=&#34;podsc&#34;&gt;为普通用户配置密码&lt;/center&gt;

![image-20220802124905092](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20220802124905092.png)

&lt;center class=&#34;podsc&#34;&gt;为普通用户配置密码——二次确认&lt;/center&gt;

### 设置时钟时区

Eastern 美东时间

Central 北美中部

Mountain 北美山区时区

Pacific 太平洋时区

Alaska 阿拉斯加夏令时间 

Hawaii 夏威夷时区

Arizona 亞利桑时区

East Indiana 印第安纳时区

Samoa 萨摩亚时间

![image-20220802124936969](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20220802124936969.png)

&lt;center class=&#34;podsc&#34;&gt;配置时区&lt;/center&gt;

### 对磁盘分区

此步骤磁盘进行分区。这里选择“手动”选项

![image-20220802125018955](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20220802125018955.png)

&lt;center class=&#34;podsc&#34;&gt;选择分区模式&lt;/center&gt;

选择手动进行划分为所需的分区。

![image-20220802125033249](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20220802125033249.png)

&lt;center class=&#34;podsc&#34;&gt;选择硬盘&lt;/center&gt;

创建新的分区表

![image-20220802125051231](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20220802125051231.png)

&lt;center class=&#34;podsc&#34;&gt;创建分区表&lt;/center&gt;

选择空闲的空间进行分区

![image-20220802125208239](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20220802125208239.png)

&lt;center class=&#34;podsc&#34;&gt;选择空闲空间&lt;/center&gt;

创建一个新分区

![image-20220802125220023](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20220802125220023.png)

&lt;center class=&#34;podsc&#34;&gt;创建一个新分区&lt;/center&gt;

为/boot划分分区

![image-20220802170825990](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20220802170825990.png)

&lt;center class=&#34;podsc&#34;&gt;为/boot划分分区&lt;/center&gt;

![image-20220802170914699](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20220802170914699.png)

&lt;center class=&#34;podsc&#34;&gt;最终划分的分区&lt;/center&gt;

这里选No就行，提示是指不使用swap分区，No就是继续，Yes将返回分区页面

![image-20220802171300600](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20220802171300600.png)

&lt;center class=&#34;podsc&#34;&gt;对于swap分区的提示&lt;/center&gt;

创建新分区需要格式化，当前的分区将会被删除，如果是新磁盘选择Yes格式化分区

![image-20220802171406223](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20220802171406223.png)

&lt;center class=&#34;podsc&#34;&gt;确认格式化，进行分区&lt;/center&gt;

### Base System安装

这里等待安装基础系统

![image-20220802171427005](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20220802171427005.png)

&lt;center class=&#34;podsc&#34;&gt;确认格式化，进行分区&lt;/center&gt;

几分钟后， 安装后会弹出一个界面，这里会扫描其他的media，这里因为没有，选择No就行。

![image-20220802171719092](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20220802171719092.png)

&lt;center class=&#34;podsc&#34;&gt;扫描其他媒介&lt;/center&gt;

### 离线安装

会扫描安装的媒介，这里也有提示，如果没有额外的媒介，可以跳过该步骤

![image-20220802175852901](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20220802175852901.png)

&lt;center class=&#34;podsc&#34;&gt;扫描其他媒介&lt;/center&gt;



![image-20220802175829172](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20220802175829172.png)



配置网络镜像，建议配置下，如果不需要No即可

![image-20220802180008399](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20220802180008399.png)

&lt;center class=&#34;podsc&#34;&gt;配置网络镜像&lt;/center&gt;

接下来会弹出一个界面，请选择“Debian镜像国家”。这个是配置镜像地址的，选择自己的国家和镜像站即可

![image-20220802171823823](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20220802171823823.png)

&lt;center class=&#34;podsc&#34;&gt;选择镜像国家&lt;/center&gt;

这里选择的是中国和中科大镜像

![image-20220802171903846](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20220802171903846.png)

&lt;center class=&#34;podsc&#34;&gt;选择镜像地址&lt;/center&gt;

配置HTTP代理，不选择跳过

![image-20220802171929992](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20220802171929992.png)

&lt;center class=&#34;podsc&#34;&gt;配置http代理&lt;/center&gt;

如果选择网络安装，到这步骤时安装程序现在将在选择相关的。首先，选择离您所在国家最近的位置。Debian 镜像位置和域后检索剩余的文件

这里提示有一个匿名调查，这里选No即可

![image-20220802180137667](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20220802180137667.png)

&lt;center class=&#34;podsc&#34;&gt;匿名调查&lt;/center&gt;

### 根据要求调整安装

在检索过程中，系统将提示需要自行选择以下预定义软件中的一个或多个。最小化安装仅选择基础系统与SSH即可

![image-20220802180225093](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20220802180225093.png)

&lt;center class=&#34;podsc&#34;&gt;安装组件选择&lt;/center&gt;

接下来等待安装即可

![image-20220802180343919](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20220802180343919.png)

&lt;center class=&#34;podsc&#34;&gt;安装过程&lt;/center&gt;

&gt; Notes：选择了DVD ISO将离线完成安装，如果使用了CD ISO，将从互联网上检索包并安装，这个时间将很长。

其中会提示一个引导按章，直接Yes即可

![image-20220802180753937](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20220802180753937.png)

到了这里即将安装完成

![image-20220802180829194](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20220802180829194.png)

&lt;center class=&#34;podsc&#34;&gt;到了这里即将安装完成&lt;/center&gt;

### 完成Debian11最小化安装

看到到这里已经完成了安装，按“Continue”继续重启后即可

![image-20220802180902242](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20220802180902242.png)

&lt;center class=&#34;podsc&#34;&gt;完成安装&lt;/center&gt;

![image-20220802180923668](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20220802180923668.png)

&lt;center class=&#34;podsc&#34;&gt;看到系统的引导界面&lt;/center&gt;

Enjoy 👏👏
