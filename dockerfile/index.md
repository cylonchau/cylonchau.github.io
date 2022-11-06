# Dockerfile

[TOC]

## 一、使用前提

通用镜像未必与应用程序和配置是符合我们需要的。

### 1.1 常见镜像制作方式
常见制作镜像方式有两种
* 基于容器
* 基于镜像制作：编辑一个Dockerfile，而后根据此文件制作；

## 二、Dockerfile概述

Dockerfile只是构建Docker镜像的源代码，docker可以通过读取Dockerfile中的指令自动构建图像。Dockerfile是一个文本文档，其中包含用户可以在命令行上调用以组合图像的所有命令。用户可以使用 &lt;font color=&#34;#f8070d&#34; size=3&gt;`docker build`&lt;/font&gt; 创建连续执行多个命令行指令的自动构建。

### 2.1 Dockerfile的工作模式

基于Dockerfile制作镜像时，需在专用目录放置Dockerfile文件，并且文件首字母必须大写。基于Dockerfile中打包的文件必须奇基于工作目录往下走的路径。在打包镜像时，&lt;font color=&#34;#f8070d&#34; size=3&gt;`.dockeringore`&lt;/font&gt; 文件本身与所有包含在 &lt;font color=&#34;#f8070d&#34; size=3&gt;`.dockeringore`&lt;/font&gt; 文件中的路径，都不被打包进去。在Dockerfile制作环境为底层镜像启动容器时所能够提供的环境。


### 2.2 环境变量替换

制作镜像中还可以使用环境变量。环境变量（使用ENV语句声明）也可以在某些指令中使用，因为`Dockerfile`环境变量在 `Dockerfile` 中以 &lt;font color=&#34;#f8070d&#34; size=3&gt;`$variable_name`&lt;/font&gt; 或&lt;font color=&#34;#f8070d&#34; size=3&gt;`${variable_name}`&lt;/font&gt;标记。

语法还支持一些标准`bash修饰符

* &lt;font color=&#34;#f8070d&#34; size=3&gt;`${variable:-word}`&lt;/font&gt; 表示如果设置了变量，那么结果将是该值。如果未设置变量，那么word将是结果。
* &lt;font color=&#34;#f8070d&#34; size=3&gt;`${variable&#43;word}`&lt;/font&gt; 表示如果设置了变量，则word将是结果，否则结果为空字符串。

## 三、Dockerfile指令说明

***
**&lt;font color=&#34;#f8070d&#34; size=3&gt;特别说明：Dockerfile中每一条指令都会生成一个新的镜像层。&lt;/font&gt;**
***

### FROM
FROM指令（最重要的一个），&lt;font style=&#34;background:#ffff00;&#34; size=2&gt;必须为Dockerfile文件开篇的第一个非注释行&lt;/font&gt;，用于为镜像文件构建过程指定基准镜像，后续的指令运行于此基准镜像所提供的运行环境。基准镜像可以是任何可用镜像文件，默认情况下，`docker build`会在docker主机上查找指定的镜像文件，在其不存在时，则会从 &lt;font color=&#34;#f8070d&#34; size=3&gt;`Docker Hub Registry`&lt;/font&gt; 上拉取所需的镜像文件。如果找不到指定的镜像文件，`docker build` 会返回一个错误信息

&gt; **Syntax:**

```sh
FROM repository[:tag]
```

```sh
FROM registry/repository[:tag]
```
```bash
FROM resository@[digest]  #←相同名称时，可以使用resository@镜像hush码指定镜像。
```

| 参数           | 说明-                                                        |
| -------------- | ------------------------------------------------------------ |
| **reposotiry** | 某一个镜像的仓库，如redis镜像仓库。                          |
| **registry**   | docker镜像仓库，如docker hub，docker registry包含很多reposotiry，如nginx php tomcat等。不指定registry，默认从docker hub下载。 |
| **tag**        | image的标签，为可选项，省略时默认为latest。                  |

### MAINTANIER(depreacted)

MAINTANIER（depreacted）用于让Dockerfile制作者提供本人的详细信息。Dockerfile并不限制MAINTAINER指令可在出现的位置，但推荐将其放置于FROM指令之后。
&gt; **Syntax**

```sh
MAINTAINER &lt;authtor&#39;s detail&gt;  
```
&lt;author&#39;s detail&gt; 可是任何文本信息，但约定俗成地使用作者名称及邮箱地址
```sh
MAINTAINER &#34;lc &lt;12399.com@gmail.com&gt;&#34;
```

### LABLE
LABLE可以提供Key value信息，比MAINTANIER具有更宽泛的使用领域。LABLE让用户提供格式各样的元数据，都是键值格式。如果在LABEL值中包含空格，请使用引号和反斜杠。image可以有多个tag。您可以在一行上指定多个tag。docker17&#43;增加此指令。

&gt; **Syntax:**

```sh
LABEL &lt;key&gt;=&lt;value&gt; &lt;key&gt;=&lt;value&gt; &lt;key&gt;=&lt;value&gt;...
```

```sh
LABEL maintainer=&#34;lc &lt;12399.com@gmail.com&gt;&#34;
```

### COPY

用于从宿主机工作目录将文件复制至到镜像的文件系统中。

&gt; **Syntax:**

```sh
COPY src...dest
COPY &#34;src&#34;....&#34;dest&#34;
```

| 参数 | 说明                                                         |
| ---- | ------------------------------------------------------------ |
| src  | 要复制的源文件或目录，支持使用通配符。                       |
| dest | 目标路径，即正在创建的image的文件系统路径；建议为dest使用绝对路径，否则，COPY指定则以WORKDIR为其起始路径。 |

***
注意：在路径中有空白字符时，通常使用第二种格式。
***
文件复制准则
* `src` 必须是build上下文中的路径，不能是其父目录中的文件。
* 如果src是目录，则其内部文件或子目录会被递归复制，但src目录自身不会被复制。 等同于 &lt;font color=&#34;#f8070d&#34; size=3&gt;`cp a/*`&lt;/font&gt;而不是 &lt;font color=&#34;#f8070d&#34; size=3&gt;`cp -r a`&lt;/font&gt;
* 如果指定了多个src，或在src中使用了通配符，则dest必须是一个目录，且必须以&lt;font color=&#34;#f8070d&#34; size=3&gt;`/`&lt;/font&gt;结尾。
* 如果dest事先不存在，它将会被自动创建，这包括其父目录路径。

### ADD

ADD指令类似于COPY指令，ADD支持使用TAR文件和URL路径。

add官方解释：[ADD](https://docs.docker.com/engine/reference/builder/#add)

如果src是以

&gt; **Syntax:**

```sh
ADD src dest
ADD [&#34;src&#34;…&#34;dest&#34;]
```
操作准则同COPY指令
* 如果 `&lt;src&gt;` 为URL且 `&lt;dest&gt;` 不以 &lt;font color=&#34;#f8070d&#34; size=3&gt;`/`&lt;/font&gt; 结尾，则 `&lt;src&gt;` 指定的文件将被下载并直接被创建为dest；如果dest以&lt;font color=&#34;#f8070d&#34; size=3&gt;`/`&lt;/font&gt;结尾，则文件名URL指定的文件将被直接下载并保存为 &lt;font color=&#34;#f8070d&#34; size=3&gt;`dest/filename`&lt;/font&gt;

* 如果src是一个本地系统上的可识别的压缩格式（identity，gzip，bzip2或xz）的本地 tar存档，则将其解压缩为目录。，其行为类似于 &lt;font color=&#34;#f8070d&#34; size=3&gt;`tar -xf`&lt;/font&gt; 命令；然而，从URL远程网址不会自动解压。

* 如果src有多个，或其间接或直接使用了通配符，则dest必须是一个以 &lt;font color=&#34;#f8070d&#34; size=3&gt;`/`&lt;/font&gt; 结尾的目录路径；如果dest不以 &lt;font color=&#34;#f8070d&#34; size=3&gt;`/`&lt;/font&gt; 结尾，则其被视作一个普通文件，src的内容将被直接写入到dest。
***
&lt;font color=&#34;#0215cd&#34; size=3&gt;Dockerfile中每一条指令都会生成一个新的镜像层。尽量避免很多指令&lt;/font&gt;
***
### WORKDIR

用于为Dockerfile中所有的 &lt;font color=&#34;#f8070d&#34; size=3&gt;`RUN`&lt;/font&gt;、&lt;font color=&#34;#f8070d&#34; size=3&gt;`CMD`&lt;/font&gt;、&lt;font color=&#34;#f8070d&#34; size=3&gt;`ENTRYPOINT`&lt;/font&gt;、&lt;font color=&#34;#f8070d&#34; size=3&gt;`COPY`&lt;/font&gt; 和 &lt;font color=&#34;#f8070d&#34; size=3&gt;`ADD`&lt;/font&gt; 指定设定工作目录
&gt; **Syntax:**

```sh
WORKDIR dirpath  
```
在Dockerfile文件中，WORKDIR指令可出现多次，其路径也可以为相对路径，不过，其是相对此前一个WORKDR指令指定的路径。另外，WORKDIR也可满用由ENV指定定义的变量

例如

```sh
WORKDIR /var/log
WORKDIR $STATEPATH  
```

### VOLUME
用于在image中创建一个挂载点目录，以挂载Docker host上的卷或其它容器上的卷，在Dockerfile中的镜像自动指定VOLUME时，一般只能指定挂载点，不能指定宿主机文件。被称之为docker管理的卷。

&gt; **Syntax:**

```sh
VOLUME mountpoint
```

```sh
VOLUME [&#34;mountpoint&#34;]
```

如果挂载点目录路径下此前在文件存在，docker run命令会在卷挂载完成后将此前的所有文件复制到新挂载的卷中。


### EXPOSE
用于为容器打开指定要监听的端口以实现与外部通信，写在文件中的端口暴露并不会直接暴露，当&lt;font color=&#34;#f8070d&#34; size=3&gt;`docker run -P`&lt;/font&gt;时不用声明端口，会自动读取镜像中指定要暴露的端口，动态分配到宿主机端口上。

&gt; **Syntax:**

```sh
EXPOSE port[/protocol] [port[/protocol]  
```
protocol用于指定传输层协议，可为tcp或udp二者之一，默认为TCP协议。

EXPOSE指令可一次指定多个端口，例如

```sh
EXPOSE 11211/udp 11211/tcp  
```


### ENV
用于为镜像定义新需的环境变量，并可被Dockerfile文件中位于其后的其它指令（如ENV、ADD、COPY等）所调用格式为 &lt;font color=&#34;#f8070d&#34; size=3&gt;`$variable_name`&lt;/font&gt; 或 &lt;font color=&#34;#f8070d&#34; size=3&gt;`${variable_name}`&lt;/font&gt;。在Dockerfile中所定义的所有环境变量，是可以在启动容器后直接在容器中使用的变量。在运行容器时更改ENV并不会影响`docker build`的值。

&gt; **Syntax:**

```sh
ENV key value
```

```sh
ENV key1=value1 key2=value2....    
```
第一位格式中。key之后的所有内容均会被视作其value的组成部分，因此，一次只能设置一个变量。第二种格式可用一次设置多个变量，每个变量为一个 &lt;font color=&#34;#f8070d&#34; size=3&gt;`key=value`&lt;/font&gt; 的键值对，如果value中包含空格，可以以反斜线（\）进行转义，也可通过对value加引号递行标识；另外，反斜线也可用于续行。

在定义多个变量时，建议使用第二种方式，以便在同一层中完成所有功能。

![image](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/085a742e.png)

运行命令。

### RUN
用于指定 &lt;font color=&#34;#f8070d&#34; size=3&gt;`dodker build`&lt;/font&gt; 过程中运行的程序，其可以是任何命令。RUN可以运行多次的，如果多个命令彼此间有关联关系，建议在一条RUN中将多个Command写进来。

&gt; **Syntax:**  

```sh
RUN command
RUN [&#34;executable&#34;，&#34;param1&#34;，&#34;param2&#34;]  
```
第一种格式中，command通常是一个shell命令，且以 &lt;font color=&#34;#f8070d&#34; size=3&gt;`/bin/sh -c`&lt;/font&gt; 来运行它，这意味着此进程在容器中的PID不为1，不能接枚Unix信号，因此，当使用 &lt;font color=&#34;#f8070d&#34; size=3&gt;`docker stop container `&lt;/font&gt; 命令停止容器时，此进程接收不到SIGTERM信号；

第二种语法格式中的参数是一个JSON格式的数组，其中executable为要运行的命令，后面的 &lt;font color=&#34;#f8070d&#34; size=3&gt;`paramN`&lt;/font&gt; 为传递给命令的选项或参数；然而，此种格式指定的命令不会以 &lt;font color=&#34;#f8070d&#34; size=3&gt;`/bin/sh -c`&lt;/font&gt; 来发起，因此常见的shell操作如变量替换以及通配符（，*等）替换将不会进行；不过，如果要运行的命令依赖于此shell特性的话，可以将其替换为奏似下面的格式。
```sh
RUN [&#34;/bin/bash&#34; , &#34;-c&#34;，&#34;executable&#34;, &#34;param1&#34;]  
```

CMD是在镜像运行为容器时没有指定默认运行命令时所运行的命令。
RUN是在基于Dockerfile构建镜像时要运行的命令。将在docker build中运行。

### CMD
类似于RUN指令，CMD指令也可用于运行任何命令或应用程序，不过，二者的运行时间点不同。RUN指令运行于镜像文件构建过程中，而CMD指令运行于基于Dockerfile构建出的新映像文件启动一个容器时

  CMD指令的首要目的在于为启动的容器指定默认要运行的程序，且其运行结束后，容器也将终止；不过，CMD指定的命令其可以被&lt;font color=&#34;#f8070d&#34; size=3&gt;`docker run`&lt;/font&gt;的命令行选项所覆盖。&lt;font style=&#34;background:#ffff00;&#34; size=2&gt;在Dockerfile中可以存在多个CMD指令，但仅最后一个会生效&lt;/font&gt;。

&gt; **Syntax:**

```sh
CMD command
```

```sh
CMD [&#34;executable&#34; , &#34;param1&#34; , &#34;param2&#34;] 
```

```sh
CMD [&#34;param1&#34; , &#34;param2&#34;]  
```
* 使用第一种方式默认使用bin/sh -c
* 前两种语法格式的意义同RUN
* 第三种则用于为ENTRYPOINT指令提供默认参数

  
### ENTRYPOINT
在&lt;font color=&#34;#f8070d&#34; size=3&gt;`docker run`&lt;/font&gt;时明明指定的运行命令为nginx，但是可以执行&lt;font color=&#34;#f8070d&#34; size=3&gt;`docker run -it --rm busybox ls /`&lt;/font&gt;，这表示了更改了镜像中默认要运行的程序。没有运行默认程序，转而运行了指定的命令。 

对于自定义的镜像而言，默认在运行容器时运行的命令是可以被覆盖的。而不允许在运行命令是改变默认命令CMD就无法完成，而&lt;font color=&#34;#f8070d&#34; size=3&gt;`ENTRYPOINT`&lt;/font&gt;可以做到


类似CMD指令的功能，用于为容器指定默认运行程序，从而使得容器像是一个单独的可执行程序。但是与CMD不同的是，。&lt;font style=&#34;background:#ffff00;&#34; size=2&gt;由ENTRYPOINT启动的程序不会被docker run命令行指定的参数所覆盖&lt;/font&gt;，而且，这些命令行参数会被当作参数传递给ENTRYPOINT指定的程序。当CMD和ENTRYPOINT同时存在时，CMD的内容会当做参数传给ENTRYPOINT。不过，docker run命令的--entrypoint选项的参教可覆盖ENTRYPOINT指令指定的程序。

&gt; **Syntax:**
```sh
ENTRYPOINT command
ENTRYPOINT [&#34;executable&#34;，&#34;paraml&#34;，&#34;param2&#34;]  
```

docker run命令传入的命令参数会覆盖CMD指令的内容并且附加到ENTRYPOINT命令最后做为其参数使用，Dockerfile文件中也可以存在多个ENTRYPOINT指令，但仅有最后一个会生效

当同时CMD与ENTRYPOINT，在运行容器时传入参数会覆盖CMD，除非使用`--entrypoint选项`否则ENTRYPOINT不能够被覆盖。

```sh
CMD [&#34;/bin/httpd&#34; , &#34;-f&#34; , &#34;-h ${WEB00C_ROOT}&#34; ]
ENTRYPOINT [&#34;/bin/sh&#34;,&#34;-c&#34;]  
```

**为什么非要同时使用CMD与ENTRYPOINT？**

1. 多数情况下ENTRYPOINT是用来指定一个shell，指定一个谁用来作为启动别的进程的服务进程。在命令行中的参数会当做他的子进程来启动。这样就可以灵活传参数被shell所解析了。
2. 容器接受配置要靠环境变量，要想让应用镜像（如，nginx）在run时能够通过环境变量接受参数来决定他的配置文件（监听地址、端口、document_root），变量可以在启动容器时进行传递。

```sh
#!/bin/sh
cat &gt;/etc/nginx/conf.d/www.conf &lt;&lt; EOF 
  server{
    server name ${HOSTNAME};
    listen ${IP:-0.0.0.0}:{PORT:-80};
    root ${NGX_DOC_ROOT:-/usr/share/nginx/html};
}
EOF exec &#34;$@&#34;  
```

```sh
FROM nginx:1.14-alpine 
LABEL maintainer=&#34;lc &lt;lc.com&gt;&#34;
ENV NGX_DOC_ROOT=&#34;/data/web/html/&#34;
ADD index.html ${NGX_DOC_ROOT}
ADD entrypoint.sh /bin/
CMD [&#34;/usr/sbin/nginx&#34; , &#34;-g&#34; , &#34;daemon off;&#34; ]
ENTRYPOINT [&#34;/bin/entrypoint.sh&#34;]
```

### USER
用于指定运行image时的或运行Dockerfile中任何RUN、CMD或ENTRYPOINT指令指定的程序时的用户名或UID，默认情况下，container的运行身份为root用户

&gt; **Syntax:**

```sh
USER UID|UseName  
```
需要注意的是，UID可以为任意教字，但实践中必须为 &lt;font color=&#34;#f8070d&#34; size=3&gt;`/etc/passwd`&lt;/font&gt; 中某用户的有效UID，否则，&lt;font color=&#34;#f8070d&#34; size=3&gt;`docker run`&lt;/font&gt; 命令将运行失败

在基于某个镜像启动容器后，只要容器没转向后台（没停止），这个容器就不会停止。在docker引擎在判定容器健康与否并不是主进程能否提供服务，而仅仅判断进程是否运行。因此docker判断机制并不是真正意义上判断主进程的是否健康，需要其他工具来辅助确定。

### HEALTHCHECK
HEALTHCHECK指令定义一个command，&lt;font color=&#34;#f8070d&#34; size=3&gt;`CMD`&lt;/font&gt; 为固定关键词，&lt;font color=&#34;#f8070d&#34; size=3&gt;`CMD`&lt;/font&gt; 后指定一个命令，这个命令用于检查容器主进程工作状态健康与否。即使主进程仍在运行，这也可以检测到陷入无限循环且无法处理新连接的Web服务器等情况。

HEALTHCHECK指令有

```sh
HEALTHCHECK [OPTIONS] CMD command # 通过在容器内运行命令来检查容器运行状况
HEALTHCHECK NONE # 禁用任何的健康状态检查，包括默认的健康状态检测机制
```

可以在CMD之前出现的选项：


| 参数                | 说明-                                                        |
| ------------------- | ------------------------------------------------------------ |
| \-\-interval &amp;nbsp; | 间隔 s秒、m分钟、h小时，default:30s。                        |
| \-\-timeout         | 执行command需要时间，比如curl一个地址，如果超过timeout秒则认为超时是错误的状态，此时每次健康检查的时间是timeout&#43;interval秒。default:30s。 |
| \-\-start-period    | 在启动容器是，对主进程自我初始化较慢的情况下，default:0s。17.05引入 |
| \-\-retries=N       | 失败次数，default:3。                                        |
|                     |                                                              |

当指定了健康检测状态命令，检测请求发出时，响应值为如下3种情况：

* 0: 成功，容器健康且随时可用。
* 1: 不健康，容器无法正常工作。 
* 2: 预留值，无意义，可以自行定义。

|HEALTHCHECK--start-period=3s CMD wget-0--q http://${IP:-0.0.0.0}:10080/


For example 
```sh
  HEALTHCHECK--interval=5m --timeout=3s \
  CMD curl -f http://locdlhost/ || exit 1  
```


### SHELL

SHELL指令允许覆盖用于shell命令形式的默认shell。在Linux上的默认shell是 &lt;font color=&#34;#f8070d&#34; size=3&gt;`[&#34;/bin/sh&#34;,&#34;-c&#34;]`&lt;/font&gt; , 在Windows上是 &lt;font color=&#34;#f8070d&#34; size=3&gt;`[&#34;cmd&#34; , &#34;/S&#34; , &#34;/C&#34;]`&lt;/font&gt; 。SHELL指令必须以JSON格式写入Dockerfile。

SHELL指令可以多次出现。每个SHELL指令覆盖先前的SHELL指令，并影响所有后续指令。

&gt; **Syntax:**
```sh
SHELL [&#34;executable&#34;,&#34;parameters]  
```

### STOHSIGNAL
STOPSIGNAL指令设置将发送到容器的系统调用信号，以退出。此信号可以是与内核的系统调用表中的位置匹配的有效无符号数，例如9，或SIGNAME格式的信号名，例如SIGKILL。
语法：&lt;font color=&#34;#f8070d&#34; size=3&gt;`STOPSIGNAL signal`&lt;/font&gt;

### ARG
ARG指令使用 &lt;font color=&#34;#f8070d&#34; size=3&gt;`--build-arg varname=&#39;value&#39;`&lt;/font&gt; 标志定义一个变量，用户可以在使用&lt;font color=&#34;#f8070d&#34; size=3&gt;`docker build`&lt;/font&gt;命令在构建时将其传递给构建器。

此功能使得一个Dockerfile能够适用于较多的不同场景，尤其是应用程序版本变换时。直接传参数就能确定应该基于Dockerfile制作哪个版本。&lt;font style=&#34;background:#fee904;&#34; size=2&gt;如果用户指定了未在Dockerfile中定义的构建参数，则构建会输出警告&lt;/font&gt;。

&gt; **Syntax:**

```sh
ARG name = default value
```

* Dockerfile中可以包括一个或多个ARG指令。
* ARG指令可以选择性地包括默认值：
```sh
ARG version = 1.14
ARGuser = mageedu  
```


### ONBUILD

用于在Dockerfile中定义一个触发器。Dockerfile用于build映像文件，此映像文件亦可作为base image被另一个Dockerfile用作FROM指令的参数，并以之构建新的映像文件

在后面的这个Dockerfile中的FROM指令在build过程中被执行时，将会“触发”创建其base image的Dockerfile文件中的ONBUILD指令定义的触发器

&gt; **Syntax:**

```sh
ONBUILD INSTRUCTION  
```
**注意：**

* 尽管任何指令都可注册成为触发器指令，但ONBUILD不能自我嵌套，且不会触发FROM和MAINTAINER指令。
* 使用包含ONBUILD指令的Dockerfle构建的镜像应该使用特殊的标签，例如ruby：2.0-onbuild
* 在ONBUILD指令中使用ADD或COPY指令应该格外小心，因为新构建过程的上下文在缺少指定的源文件时会失败。

  
## 四、构建php环境镜像

```bash
FROM  centos:6
MAINTAINER   lc
RUN  yum install -y httpd php php-gd php-mysql mysql mysql-server
ENV   MYSQL_ROOT_PASSWORD 123456
RUN   echo &#34;&lt;?php phpinfo()?&gt;&#34; &gt; /var/www/html/index.php
ADD   start.sh /start.sh
RUN  chmod &#43;x /start.sh
ADD   https://cn.wordpress.org/wordpress-4.7.4-zh_CN.tar.gz /var/www/html
COPY   wp-config.php /var/www/html/wordpress
VOLUME   [&#34;/var/lib/mysql&#34;]
CMD   /start.sh
EXPOSE  80 3306
```

## 五、构建java环境镜像

```sh
FROM centos

MAINTAINER lc
ADD jdk-8u144-linux-x64.tar.gz /usr/local
ENV JAVA_HOME=/usr/local/jdk1.8.0_144
ADD apache-tomcat-8.5.32.tar.gz /usr/local/
RUN mv /usr/local/apache-tomcat-8.5.32 /usr/local/tomcat
WORKDIR /usr/local/tomcat
ENTRYPOINT [&#34;bin/catalina.sh&#34;,&#34;run&#34;]
EXPOSE 8080  
```

## 六、构建ssh环境镜像

```bash
FROM centos
MAINTAINER zhangsan
ENV PWD 123
RUN yum install openssh openssh-server openssh-clients -y
RUN echo $PWD|passwd --stdin root
RUN ssh-keygen -t dsa -f /etc/ssh/ssh_host_dsa_key
RUN ssh-keygen -t rsa -f /etc/ssh/ssh_host_rsa_key
CMD [&#34;/usr/sbin/sshd&#34;,&#34;-D&#34;]  
```

#### 6.1 测试ssh镜像


&gt; **systemd启动**

构建完镜像使用systemctl启动发现没有sshd进程
```sh
[root@cae0d8aa0b0c /]# ps
   PID TTY          TIME CMD
     1 pts/0    00:00:00 bash
    18 pts/0    00:00:00 ps
[root@cae0d8aa0b0c /]# systemctl start ssh
Failed to get D-Bus connection: Operation not permitted
```

此问题原因：systemd服务没有启动无法使用systemd启动

[Failed to get D-Bus connection: No connection to service manager](https://github.com/moby/moby/issues/7459)

故启动时一般使用 `CMD [&#34;/usr/sbin/sshd&#34;,&#34;-D&#34;]`
