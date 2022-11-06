# jenkins在windows平台自动化构建代码 


jenkins服务端：centos6.8

客户端：windows server2012 windows10

工具：cwRsync

**注：复制为jenkins工作目录到网站目录，无需服务端。**

## 配置安装slave端

所用的插件：[Copy Data To Workspace Plugin](https://wiki.jenkins.io/display/JENKINS/Copy&#43;Data&#43;To&#43;Workspace&#43;Plugin)

### 配置windows节点

\1. 主界面-&gt;【系统管理】-&gt;【管理节点】-&gt;【新建节点】，进行节点的添加：

![image-20221025230809664](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20221025230809664.png)

![image-20221025230814531](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20221025230814531.png)

![image-20221025230819220](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20221025230819220.png)

\2. 输入节点名称，选择【Permanent Agent】。如果添加过slave的话会出现【复制现有节点】操作

![image-20221025230830765](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20221025230830765.png)

\3. 配置节点的详细信息

![image-20221025230848946](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20221025230848946.png)

此处配置需要注意的有以下几个方面

- 【# of executors】：建议不要超过CPU核心数，一般不要写特别大。

- 【远程工作目录】：master将代码库中的代码复制到slave时，存放的临时目录，如slave的daemon服务也会放在此目录。一个job一个文件夹。

- 【用法】：选择【**只允许运行绑定到这台机器的Job**】，此模式下，Jenkins只会构建哪些分配到这台机器的Job。这允许一个节点专门保留给某种类型的Job。例如，在Jenkins上连续的执行测试，你可以设置执行者数量为1，那么同一时间就只会有一个构建，一个实行者不会阻止其它构建，其它构建会在另外的节点运行。

- 【启动方式】：选择【**Launch agent via Java Web Start**】，以windows服务的方式启动，这个为最好配置的。注意：2.x版本的默认没有这个选项，需要单独开启。

\4. 配置slave端并且添加至windows服务

在点击保存后，在node列表中会存在此列表默认是未连通状态

![image-20221025230936940](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20221025230936940.png)

点击进入详情页面会提示slave端的安装方法，此处讲解下载文件方式。

【Launch】：浏览器下载文件方式

【Run from agent command line】：从远端代理命令运行

![image-20221025230952380](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20221025230952380.png)

**注意：这是java服务，每个slave端必须安装jdk后才可运行。**

```xml
&lt;jnlp codebase=&#34;http://10.0.0.11:8080/jenkins/computer/test/&#34; spec=&#34;1.0&#43;&#34;&gt;
&lt;information&gt;
    &lt;title&gt;Agent for test&lt;/title&gt;
    &lt;vendor&gt;Jenkins project&lt;/vendor&gt;
    &lt;homepage href=&#34;https://jenkins-ci.org/&#34;/&gt;
&lt;/information&gt;
&lt;security&gt;
    &lt;all-permissions/&gt;
&lt;/security&gt;
&lt;resources&gt;&lt;j2se version=&#34;1.8&#43;&#34;/&gt;
    &lt;jar href=&#34;http://10.0.0.11:8080/jenkins/jnlpJars/remoting.jar&#34;/&gt;
&lt;/resources&gt;
&lt;application-desc main-class=&#34;hudson.remoting.jnlp.Main&#34;&gt;
    &lt;argument&gt;c55442e04b03c2fc721ec718b70646c234b4c79a678ff10ccadc59541dbb843&lt;/argument&gt;
    &lt;argument&gt;test1&lt;/argument&gt;
    &lt;argument&gt;-workDir&lt;/argument&gt;
    &lt;argument&gt;d:\jenkins&lt;/argument&gt;
    &lt;argument&gt;-internalDir&lt;/argument&gt;
    &lt;argument&gt;remoting&lt;/argument&gt;
    &lt;argument&gt;-url&lt;/argument&gt;
    &lt;argument&gt;http://10.0.0.11:8080/jenkins/&lt;/argument&gt;
&lt;/application-desc&gt;&lt;/jnlp&gt;
```

**注意：每个slave的内容都不一样至。多个slave需要多次下载或修改此内容**

![image-20221025231027942](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20221025231027942.png)

安装出现如下错误的原因，没有权限，使用管理员方式运行。

![image-20221025231041933](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20221025231041933.png)

这种文件右键没有管理员方式运行的菜单，打开【任务管理器】-&gt;【运行】-&gt;【以管理员方式运行】

![image-20221025231052705](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20221025231052705.png)

卸载系统服务方式:

```a
sc delete jenkinsslave-c__jenkins
```

安装完成后slave设置的远端目录会生成如下文件

![image-20221025231125859](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20221025231125859.png)

返回master的节点列表里，发现此处已经连接上了。

![image-20221025231139284](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20221025231139284.png)

## 新建工程

选择自由构建方式。

![image-20221025231156436](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20221025231156436.png)

【Restrict where this project can be run】：限制运行此项目的节点为刚才设置node时标签填写的windows。

![image-20221025231211038](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20221025231211038.png)

下载安装插件后会出现此选项。实测，填写路径没什么卵用。

![image-20221025231220377](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20221025231220377.png)

此处选择执行windows命令

&gt;**注：此处存在以下问题。**
&gt;
&gt;**1、此处如果代码库中不存在此文件，或更新后此文件被删除，那么使用xcopy会存在代码库中的文件以删除，而slave node文件夹中的文件还存在。无法清除。解决方法使用rsync –delete 或执行脚本文件进行判断。**
&gt;
&gt;**2、如slave node中需要存在代码库中不存在的文件，使用rsync会将需要存在的文件删除。**
&gt;
&gt;**3、此处无环境变量，执行命令需要使用全路径，不能存在中文和空格**

```bat
$ rsync -avz ./ /cygdrive/c/test1/ --delete --exclude=.svn
 
xcopy /y /e /r ./ /cygdrive/c/test1/
```


