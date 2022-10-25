# docker Volume


对于docker来讲，作为容器运行的底层引擎，在组织及运行容器时每个容器内只运行一个程序及子程序。对于这个容器来讲，启动时依赖于
底层镜像联合挂载启动而成。
底层能够存储此类分层构建并联合挂载镜像的文件系统。最上层构建读写层。对于此读写层来说。所有对容器的操作都保存在最上层之上。而下层内容的操作需要使用写时复制。


Docker镜像由多个只读层叠加而成，启动容器时，Docker会加载只读镜像层并在镜像栈顶部添加一个读写层，如果运行中的容器修改了现有的一个已经存在的文件，那该文件将会从读写层下面的只读层复制到读写层，该文件的只读版本仍然存在，只是已经被读写层中该文件的副本所隐藏，此即 <font style="background:#ffff00;" size=3>写时复制（COW）</font>机制。此机制对IO较高的应用在实现持久化存储时，势必对在底层应用数据存储时性能要求较高。要想绕过使用限制，可以使用存储卷机制。


![image-20221025233608002](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20221025233608002.png)

Why Data Volume？

宿主机的主机文件系统直接与容器内部的文件系统之上的某一访问路径建立绑定关系。

在宿主机上目录和容器内文件系统建立绑定关系的目录相对于容器来讲被称为<font color="#f8070d" size=3>`volume`</font>。容器内所有有效数据都是保存在存储卷，从而脱离容器自身文件系统。当容器关闭并删除时，只要不删除与宿主机与之绑定的存储目录，就能实现数据脱离容器的生命周期而持久化。docker的存储卷默认情况下使用其所在宿主机之上的本地文件系统目录的。


1. 关闭并重启容器，其数据不受影响；但删除Docker容器，则其更改将会全部丢失
2. 存在的问题
3. 存储于联合文件系统中，不易于宿主机访问；
4. 容器间数据共享不便
5. 删除容器其数据会丢失
解决方案：“卷（volume）”
“卷”是容器上的一个或多个“目录”，此类目录可绕过联合文件系统，与宿主机上的某目录“绑定（关联）”

![image-20221025233624954](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20221025233624954.png)

在docker中如果需要动刀存储卷时，不必要手动创建，Volume于容器初始化之时即会创建，由base image提供的卷中的数据会于此期间完成复制

Volume的初衷是独立于容器的生命周期实现数据持久化，因此删除容器之时既不会删除卷，也不会对哪怕未被引用的卷做垃圾回收操作；

Data volumes
·卷为docker提供了独立于容器的数据管理机制
·可以把“镜像”想像成静态文件，例如“程序”，把卷类比为动态内容，例如“数据
"；于是，镜像可以重用，而卷可以共享；
·卷实现了“程序（镜像）”和“数据（卷）”分离，以及“程序（镜像）”和“制作镜像的主机
"分离，用户制作镜像时无须再考虑镜像运行的容器所在的主机的环境；

![image-20221025233642310](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20221025233642310.png)

Docker有两种类型的卷，每种类型都在容器中存在一个挂载点，但其在宿主机上的位置有所不同；

Bind mount volume 绑定挂载卷
在宿主机指定一个特定路径，在容器内指定一个特定路径，二者已知路径建立关联关系。

a volume that points to a user-specified location on the host file system

### Docker-managed volume docker管理卷

指定容器内的挂载点，与之关联的是宿主机的目录由`docker daemon`引擎自行创建空目录，或者使用已存在目录与存储卷路径建立关联关系。

the Docker daemon creates managed volumes in a portion of the host's file system that's owned by Docker

![image-20221025233706235](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20221025233706235.png)

### 在容器中使用Volumes

为docker run命令使用一v选项即可使用Volume

Docker-managed volume

```sh
docker run-it-name box1 -v /data busybox
docker inspect-f {{.Mounts} box1
```
查看bbox1容器的卷、卷标识符及挂载的主机目录
Bind-mount Volume

```sh
docker run-it-v HOSTDIR:VOLUMEDIR--name box2 busybox
docker inspect-f {{.Mounts}} box2
```

Sharing volumes There are two ways to share volumes between containers
多个容器的卷使用同一个主机目录，例如

```bash
$ docker run-it--namec1-v/docker/volumes/v1：/data busybox
$ docker run-it--name c2-v/docker/volumes/v1：/data busybox
```

复制使用其它容器的卷，为docker run命令使用--volumes-from选项

```sh
docker run-it--name box1 -v /docker/volumes/v1:/data busybox
docker run-it--name box2 --volumes-from box1 busybox
```

EXPOSE
用于为容器打开指定要监听的端口以实现与外部通信，并不会直接暴露，只是声明需要暴露的端口，在`docker run -P`时自动暴露端口

Syntax

```sh
EXPOSE <port> [/<protocol>] [<port>[/<protocol>]..]
```

<protocol> 用于指定传输层协议，可为wp或udp二者之一，默认为TCP协议
EXPOSE指令可一次指定多个端口，例如

```sh
EXPOSE 11211/udp 11211/tcp
```

ENV
用于为镜像定义所需的环境变量，并可被Dockerfile文件中位于其后的其它指令（如ENV、ADD、COPY等）所调用

调用格式为Svariable_name或${variable_name}

### Syntax

- ENV `<key>` `<value>`或
- ENV `<key>`=`<value>` .…

第一种格式中，<key>之后的所有内容均会被视作其 `<value>` 的组成部分，因此，一次只能设置一个变量；

第二种格式可用一次设置多个变量，每个变量为一个 `<key>=<value>` 的键值对，如果
  `<value>` 中包含空格，可以以反斜线（）进行转义，也可通过对 `<value>` 加引号进行标识；另外，反斜线也可用于续行；

定义多个变量时，建议使用第二种方式，以便在同一层中完成所有功能
