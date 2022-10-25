# tomcat使用

[toc]

## 1 tomcat介绍

Tomcat 服务器是一个免费的开放源代码的Web 应用服务器，Tomcat是Apache 软件基金会（Apache Software Foundation）的Jakarta 项目中的一个核心项目，它早期的名称为catalina，后来由Apache、Sun 和其他一些公司及个人共同开发而成，并更名为Tomcat。

### 1.1 java体系结构

1.  java程序设计语言（编程）
2.  java类文件格式
3.  java api
4.  java vm

java 纯面向对象语言

obj：以指令为中心，围绕指令组织数据

面向对象：以数据为中心，围绕数据组织指令

Tomcat核心组件：

- Catalina：servlet container在tomcat中用于实现servlet代码的被称作Catalina
- Coyote：一个http连接器，能够接受http请求并相应http请求的web服务器
- jasper：JSP Engine jsp翻译器

### 1.2 Tomcat组成部分

Tomcat支持Servlet和JSP1的规范，它由一组嵌套的层次和组件组成，一般可分为以下四类：

- 顶级组件：位于配置层次的顶级，并且彼此间有着严格的对应关系；
- 连接器：连接客户端（可以是浏览器或Web服务器）请求至Servlet容器，
- 容器：包含一组其它组件；
- 被嵌套的组件：位于一个容器当中，但不能包含其它组件；

### 1.3 tomcat常见组件

#### 1.3.1 服务器(server)

服务器(server)顶级组件：Tomcat的一个实例，通常一个JVM只能包含一个Tomcat实例；因此，一台物理服务器上可以在启动多个JVM的情况下在每一个JVM中启动一个Tomcat实例，每个实例分属于一个独立的管理端口。这是一个顶级组件。

##### server相关属性

className：用于实现此Server容器的完全限定类的名称，默认为 `org.apache.cataliana.core.StandardServer`; 

port：接收shutdown指令的端口，默认仅允许通过本机访问，默认为8005

shutdown：发往此Server用于实现关闭tomcat实例的命令字符串，默认为SHUTDOWN

#### 1.3.2 服务(service)

Service主要用于关联一个引擎和此引擎相关的连接器，每个连接器通过一个特定的端口和协议接收入站请求将其转发至关联的引擎进行处理，因此，Service要包含一个引擎（Engine）、一个或多个连接器（Connector）。
定义一个名为Catalina的Service，此名字也会产生相关的日志信息时记录在日志文件当中。

```xml
<Service name="Catalina">
```

##### Service相关属性

className：用于实现service的类名，一般都是 `org.apache.catalina.core.StandardService`

name：此服务的名称，默认为Catalina。

#### 1.3.3 连接器(connectors)

分析并接收用户请求，并把它转换成本地jsp文件代码的请求。负责连接客户端（可以是浏览器或Web服务器）请求至Servlet容器内的Web应用程序，通常指的是接收客户发来请求的位置及服务器端分配的端口。<font style="background:#fee904;" size=3> 默认端口通常是HTTP协议的8080</font>，管理员也可以根据自己的需要改变此端口。一个引擎可以配置多个连接器，但这些连接器必须使用<font style="background:#fee904;" size=3>不同的端口</font>。默认的连接器是基于HTTP/1.1的Coyote。同时，Tomcat也支持AJP、JServ和JK2连接器。

**进入Tomcat的请求可以根据Tomcat的工作模式分为如下两类：**

Tomcat作为应用程序服务器：请求来自于前端的web服务器，这可能是Apache IIS Nginx。

Tomcat作为独立服务器：请求来自与web浏览器。

Tomcat应该考虑工作情形并为相应情形下的请求分别定义好需要的连接器才能正确接收来自于客户端的请求，一个引擎可以有一个或多个连接器，以适应多种请求方式。

定义连接器可以使用多种属性，有些属性也只是适用于某种特定的连接器类型，

***一般来说，常见于server.xml中的连接器类型通常有4种：***

1.  HTTP连接器
2.  SSL连接器
3.  AJP 1.3连接器
4.  proxy连接器

```xml
<Connector port="8080" protocol="HTTP/1.1"
 	maxThreads="150" connectionTimeout="20000"
 	redirecPort="8443" />
```

##### Connector常见属性

定义连接器时可以配置的属性非常多，单通常定义HTTP连接器时必须定义的属性只有<font style="background:#fee904;" size=3>port</font>，定义AJP连接器时必须定义的属性只有<font style="background:#fee904;" size=3>protocol</font>，<font style="background:#fee904;" size=3>默认的协议为HTTP</font>。

| 属性 | 解释 |
| ---- | ---- |
|address	|指定连接器监听的地址，默认为所有地址，即0.0.0.0。|
|maxThread	|支持的最大（线程）并发连接数，默认为200。|
|port	|监听的端口，默认为0。|
|protocol	|连接器使用的协议，默认为HTTP/1.1，定义AJP协议时通常为AJP/1.3。|
|redirectPort	|如果某种连接器支持的协议是HTTP，当接收客户端发来的HTTPS请求时，则转发至此属性定义的端口。|
|connectionTimeout|	等待客户端发送请求的超时时间.单位为毫秒，默认为60000，即1分钟。|
|enableLookups|	是否通过request.getRemoteHost()进行DNS直询以获取客户端的主机名；默认为true。|
|acceptCount|	设置等待队列的最大长度：通常在tomcat所有处理线程均处于繁忙时，新发来的请求将被放置于等待队列中。|
|minSpareThreads|最小空闲进程|

**下面是定义了多个属性的SSL简介器：**

```xml
<Connector port="8443" 
 	maxThread="150" minSpareThreads="25" maxSpareThreads="75"
 	enableLookups="false" acceptCount="100" debug="0" scheme="https"
 	secure="true" 
 	clientAuth="false" sslProtocol="TLS" />
```

#### 1.3.4 引擎(Engine)

引擎是指处理请求的Servlet引擎组件，即Catalina Servlet引擎，它从HTTPconnector接收请求并响应请求。它检查每一个请求的HTTP首部信息以辨别此请求应该发往哪个host或context，并将请求处理后的结果返回的相应的客户端。严格意义上来说，容器不必非得通过引擎来实现，它也可以是只是一个容器。如果Tomcat被配置成为独立服务器，默认引擎就是已经定义好的引擎。而如果Tomcat被配置为Apache Web服务器的提供Servlet功能的后端，默认引擎将被忽略，因为Web服务器自身就能确定将用户请求发往何处。一个引擎可以包含多个host组件。

##### Engine组件:

Engine是Servlet处理器的一个实例，即servlet引擎，默认为为定义在server.xml中的Catalina。 Engine需要的defaultHost属性来为其定义一个接收所有发往非明确定义虚拟主机的请求host组件。如前面示例中定义的：

```xml
<Engine name='Catalina' defaultHost='localhost'>
```

Engine容器中可以包含Realm、Host、Listener和Valve子容器

##### engine常用属性

|属性	|说明|
|:---|:-------|
|defaultHost|	Tomcat支持基于FQDN的虚拟主机，这些虚拟主机可以在Engine容器中定义多个不同的Host组件来实现；单如果此引擎的连接器收到一个发往非明确定义虚拟主机的请求时则需要将此请求发往一个默认的虚拟主机进行处理，因此，在Engine中定义的多个虚拟主机的主机名称至少要有一个根defaultHost定义的书记名称同名。|
|name|	Engine组件的名称，用于日志和错误信息记录时区别不同的引擎。|
|jvmroute	||

**tomcat由众多组件组成，这些组件分别用于实现不同的功能。**

```xml
<server>
 	<service>
 		<connector />
 		<connector />
 		...
 		<engine>
  			<host>
 		 		<context />
 			..
 			</host>
 			..
 		</engine>
 	</service>
</server>
```

#### 1.3.5 主机(Host)

主机组件类似于Apache中的 <font style="background:#fee904;" size=3>虚拟主机</font>，但在Tomcat中只支持基于FQDN的“虚拟主机”。一个引擎至少要包含一个主机组件。
位于Engine容器中，用于接收请求并进行相应处理的主机或虚拟主机

```xml
<Host name='localhsot' appBase='webapps'
 	unpackWARs='true' autoDeploy='true'
 	xmlValidation='false' xmlNamespaceAware='false'>
</Host>
```

##### 常用属性

|属性	|说明|
|:-----|:----|
|appBase	|此host的webapps目录，即存放非归档的web应用程序的目录或归档后的WAR文件的目录路径。可以位用基于$CATALINA_HOME的相对路径。|
|autoDeploy	|在Tomcat处于运行状态时放置于appBase目录中的应用程序文件是否自动进行deploy，默认认为true。|
|unpackWars	|在启用此webapp时是否对WAR格式的归的文件先进行展开，默认为true。|

##### 虚拟主机定义示例

```xml
 <Engine name='Catalina' defaultHost='localhost'>
  	<Host name='localhost' appBase='webapps'>
 		<Context path='' docBase='ROOT' />
 		<Context path='' docBase='/web/www' reloadable='true' crossContext='true' />
 	</Host>
 </Engine>
```

##### 主机别名定义

```xml
 	<Host name='localhost' appBase='webapps'>
 	 	<Alias>web.com</Alias>
 	</Host>
```

#### 1.3.6 上下文(Context)

Context组件是  最内层次的组件，它表示Web应用程序本身。配置一个Context最主要的是指定Web应用程序的根目录，以便Servlet容器能够将用户请求发往正确的位置。Context组件也可包含自定义的错误页，以实现在用户访问发生错误时提供友好的提示信息。
context在某些意义上类似于apache中的路径别名，一个Context定义用于表示tomcat实例中的一个Web应用程序。

```xml
<Context path="/web/bbs" docBase="/data/web" />
```

在tomcat6中每一个context定义也可以使用一个单独的XML文件进行，其文件的目录为`$CATALINA_HOME/conf/<engine name>/<host name>`，可以用于Context中的XML元素有Loader，Manager，Realm，Resources、WatchedResource。


|属性|	说明|
|:-------|:--------|
|docBase|相应的web应用程序的存放位置。也可以使用相对路径，起始路径为此Context所属Host中appBase定义的路径。切记，docBase的路径名不能与相应host中appBase中定义的路径名有包含关系。如：如果appBase为deploy，而docBase决不能为deploy-bbs之类的名字。|
|path	|相对与web服务器根路径而言的URI，如果为空''，则表示为此webapp的根路径，如果context定义在一个单独的xml文件中，此属性不需要定义。|
|reloadable	|是够允许重新加载此context相关的web应用程序的类，默认为false|

## 1.4 被嵌套类(nested)组件

这类组件通常包含于容器类组件中以提供具有管理功能的服务，它们不能包含其它组件，但有些却可以由不同层次的容器各自配置。

#### 1.4.1 阀门(Valve)

用来拦截请求并在将其转至目标之前进行某种处理操作，类似于Servlet规范中定义的过滤器。Valve可以定义在任何容器类的组件中。Valve常被用来记录客户端请求、客户端IP地址和服务器等信息，这种处理技术通常被称作请求转储(request dumping)。请求转储valve记录请求客户端请求数据包中的HTTP首部信息和cookie信息文件中，响应转储valve则记录响应数据包首部信息和cookie信息至文件中。

Valve（阀门）类似于过滤器，用来拦截请求并在将其转至目标之前进行某种处理操作；它可以工作于Engine和Host/Context之间、Host和Context之间以及Context和Web应用程序的某资源之间。
Valve常被用来记录客户端请求、客户端IP地址和服务器等信息，这种处理技术通常被称作请求转储(request dumping)。请求转储valve记录请求客户端请求数据包中的HTTP首部信息和cookie信息文件中，响应转储valve则记录响应数据包首部信息和cookie信息至文件中。

一个容器内可以建立多个Valve，而且Valve定义的次序也决定了它们生效的次序。不同类型的Value具有不同的处理能力，Tomcat中实现了多种不同的Valve：

- AccessLogValve：访问日志Valve。
- ExtendedAccessValve：扩展功能的访问日志Valve。
- JDBCAccessLogValve：通过JDBC将访问日志信息发送到数据库中。
- RequestDumperValve：请求转储Valve。
- RemoteAddrValve：基于远程地址的访问控制。
- RemoteHostValve：基于远程主机名称的访问控制。
- SemaphoreValve：用于控制Tomcat主机上任何容器上的并发访问数量。
- JvmRouteBinderValve：在配置多个tomcat为以Apache通过mod_proxy或mod_jk作为前段的集群架构中，当期望停止某节点时，可以通过此valve将用记请求定向至备用节点，使用civalve必须使
- JvmRouteSessionIDBinderListener。
- ReplicationValve：专用于Tomcat集群架构中，可以在某个请求的session信息发生更改时触发session数据在各节点间进行复制。
- SingleSignOn：将两个或多个需要对用户进行认证webapp在认证用户时连接在一起，即一次认证即可访问所有连接在一起的webapp。
- ClusterSingleSingOn：对SingleSignOn的扩展，专用于Tomcat集群当中，需要结合ClusterSingleSignOnListener进行工作。
- RemoteHostValve和RemoteAddrValve可以分别用来实现基于主机名称和基于IP地址的访问控制，控制本身可以通过allow或deny来进行定义，这有点类似与Apache httpd的访问控制功能：如下面的valve则实现了仅允许本机访问/probe。

```xml
<Context path='/porbe' docBase='probe'>
<Valve className='org.apache.catalina.valves.RemoteAddrValve' 
 	allow="127\.0\.0\.1" />
</Context>
```

##### 常用相关属性说明

|属性	|说明|
|:---|:-----|
|className	|相关的java实现的类名，相应于分别应该为org.apache.catalina.valves.RemoteAddrValve|
|allow|	以逗号分开的允许访问的IP地址列表，支持正则表达式，因此。点号“.”用于IP地址时需要转义，仅定义allow项时，非明企鹅allow的地址均被deny。|
|deny	|以逗号分开的禁止访问的IP地址列表，支持正则表达式。|

#### 1.4.2 日志记录器(Logger)

用于记录组件内部的状态信息，可被用于除Context之外的任何容器中。日志记录的功能可被继承，因此，一个引擎级别的Logger将会记录引擎内部所有组件相关的信息，除非某内部组件定义了自己的Logger组件。

#### 1.4.3 领域(Realm)

Realm（领域）表示分配给这些用户的用户名，密码和角色（类似于Unix组）的"数据库"。一个Realm（领域）表示一个安全上下文，它是一个授权访问某个给定Context的用户列表和某用户所允许切换的角色相关定义的列表。因此Realm就像是一个用户和组相关的数据库。定义Realm是唯一必须要提供的是classname，它是Realm的国歌 LockOutRealm：提供锁定功能，以便在给定时间段内出现过多的失败认证尝试时提供用户锁定机制；

- JAASRealm：基于Java Authintication and Authorization Service实现用户认证。
- JDBCRealm：通过JDBC访问某关系型数据库表实现用户认证。
- JNDIRealm：基于JNDI使用目录服务实现认证信息的获取。
- MemoryRealm：超找tomcat-user.xml文件实现用户信息的获取。
- UserDatabaseRealm：基于UserDatabase文件(通常是tomcat-user.xml)实现用户认证，它实现是一
- 完全可更新和持久有效的MemoryRealm，因此能够跟标准的MemoryRealm兼容；它通过JNDI实现；
  使用JDBC方式获取用户认证信息的配置

```xml
<Ream className="org.apache.catalina.realm.UserDatabaseRealm" resourceName="UserDatabase" />

<!-- 使用UserDatabase的配置 -->

<Ream className="org.apache.catalina.realm.UserDatabaseRealm" debug="99"
 	driverName="org.gjt.mm.mysql.Driver"
 	connectionUrl="jdbc:mysql//localhost/authority"
 	userTable="test" userNameCol="user_name"
 	userCredCol="user_pass"
 	userRoleTable="user_roles" roleNameCol="role_name" />
```

### 1.5 tomcat的运行模式

#### 1.5.1 standalone

通过内置的web server(http connector)来接受客户端请求；

#### 1.5.2 proxy

由专门的web server服务客户端的http请求；

in-process：不属于同一主机；

network：不属于不同主机

每一次访问jsp文件时，jsp文件会被编译成字节码存放到磁盘，下次访问时直接加载字节码不会再访问了。

server.xml：主配置文件

context.xml：每个webapp都可以有专用的配置文件，这些配置文件通常位于webapp应用程序目录下的WEB-INF目录中，用于定义会话管理器、JDBC等：conf/context.xml是为各webapp提供默认部署相关的配置；

tomcat-user.xml用户认证的账号和密码配置文件。

catalina.policy：当使用-security选项启动tomcat实例时，会读取此配置文件来实现其安全运行策略；

catalina.properties：java属性的定义文件，用于设定类加载器路径等，以及一些JVM性能相关的调优参数；

logging.properties：日志相关的配置信息；

#### 1.5.3 Tomcat连接器架构

基于Apache做为Tomcat前端的架构来讲，Apache通过mod_jk、mod_jk2或mod_proxy模块与后端的Tomcat进行数据交换。而对Tomcat来说，每个Web容器实例都有一个Java语言开发的连接器模块组件，在Tomcat6中，这个连接器是org.apache.catalina.Connector类。这个类的构造器可以构造两种类别的连接器：HTTP/1.1负责响应基于HTTP/HTTPS协议的请求，AJP/1.3负责响应基于AJP的请求。但可以简单地通过在server.xml配置文件中实现连接器的创建，但创建时所使用的类根据系统是支持APR(Apache Portable Runtime)而有所不同。

APR是附加在提供了通用和标准API的操作系统之上一个通讯层的本地库的集合，它能够为使用了APR的应用程序在与Apache通信时提供较好伸缩能力时带去平衡效用。

同时，需要说明的是，mod_jk2模块目前已经不再被支持了，mod_jk模块目前还apache被支持，但其项目活跃度已经大大降低。因此，目前更常用 的方式是使用mod_proxy模块。

***如果支持APR：***

```xml
HTTP/1.1：org.apache.cotote.http11.Http11AprProtocol
AJP/1.3：org.apache.coyote.ajp.AjpAprProtocol
```

***如果不支持APR：***

```xml
HTTP/1.1: org.apache.coyote.http11.Http11Protocol
AJP/1.3: org.apache.jk.server.JkCoyoteHandler
```

***连接器协议：***

Tomcat的Web服务器连接器支持两种协议：AJP和HTTP，它们均定义了以二进制格式在Web服务器和Tomcat之间进行数据传输，并提供相应的控制命令。

AJP(Apache JServ Protocol)协议：目前正在使用的AJP协议的版本是通过JK和JK2连接器提供支持的AJP13，它基于二进制的格式在Web服务器和Tomcat之间传输数据，而此前的版本AJP10和AJP11则使用文本格式传输数据。

HTTP协议：诚如其名称所表示，其是使用HTTP或HTTPS协议在Web服务器和Tomcat之间建立通信，此时，Tomcat就是一个完全功能的HTTP服务器，它需要监听在某端口上以接收来自于商前服务器的请求。

## 2 下载安装tomcat

***Tomcat的官方站点***

http://tomcat.apache.org/

要安装Tomcat，首先需要安装JDK。

***jdk下载地址***

http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html

***加载环境变量***

```bash
cat >/etc/profile.d/java.sh <<EOF
JAVA_HOME=/app/jdk
export PATH=$JAVA_HOME/bin:$PATH
EOF
```

***查看java版本***

```
[root@docker app]# java -version
java version "1.8.0_144"
Java(TM) SE Runtime Environment (build 1.8.0_144-b01)
Java HotSpot(TM) 64-Bit Server VM (build 25.144-b01, mixed mode)
```

### 2.1 yum安装jdk

此方法不需要对path设置

```
yum install java-1.8.0-openjdk* -y
```

### 2.2 部署tomcat

官方网站：http://tomcat.apache.org/

![image-20200730221049053](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20200730221049053.png)



![405b37ee](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/405b37ee.png)



```
[root@docker app]# tree /app/tomcat-8.5.16/ -L 1
/app/tomcat-8.5.16/
├── bin
├── conf
├── lib
├── LICENSE
├── logs
├── NOTICE
├── RELEASE-NOTES
├── RUNNING.txt
├── temp
├── webapps
└── work
```

```
/app/tomcat-8.5.16/bin/startup.sh 
```

![image-20200730221151330](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20200730221151330.png)



## 3 tomcat配置



### 3.1 tomcat的目录结构

bin：脚本及启动时用到的类

lib：类库

conf：配置文件

logs：日志文件

webapps：应用程序默认部署目录

work：工作目录

temp：临时文件目录

```bash
[root@redis tomcat]# tree -L 1 /app/tomcat/
/app/tomcat/
├── bin #<==脚本及启动时用到的类
├── conf #<==配置文件
│   ├── Catalina
│   ├── catalina.policy
│   ├── catalina.properties #<==当使用-security选项启动tomcat实例时，会读取此配置文件来实现其安全运行策略；
│   ├── context.xml #<== 每个webapp都可以有专用的配置文件，这些配置文件通常位于webapp应用程序目录下的WEB-INF目录中，用于定义会话管理器、JDBC等：conf/context.xml是为各webapp提供默认部署相关的配置
│   ├── logging.properties #<==日志相关的配置信息；
│   ├── server.xml #<==主配置文件
│   ├── tomcat-users.xml #<==用户认证的账号和密码配置文件
│   └── web.xml
├── lib #<==类库
├── logs #<==日志文件
├── temp #<==临时文件目录
├── webapps #<==应用程序默认部署目录
└── work #<==工作目录
```

### 3.2 java webapp组织结构

有特定的组织形式、层次型的目录结构：主要包含了servlet代码文件、JSP页面文件、类文件、部署描述符文件等；

/usr/local/tomcat/webapps/app1/

/webapp的根目录

- WEB-INF：当前webapp的私有资源目录，通常存放当前webapp自用的 `web.xml`

- META-INF：当前webapp的私有资源目录，通常存放当前webapp自用的 `context.xml`

- classes：此webapp的私有类

- lib：此webapp的私有类，被大包围jar格式类。

- index.jsp：webapp的主页


webapp归档格式：

- `.war`：webapp
- `.jar`：EJB的类
- `.rar`：资源适配器
- `.ear`：企业级应用程序

### 3.3 手动部署java程序

在webapp下创建特定格式的子目录。每个应用程序都会自动从tomcat conf目录下去读取web.xml和context.xml。如果没有提供默认的，就会从tomcat上继承一个公共的。如想对默认配置进行修改，建议复制web.xml到WEB-INF，context.xml到MATE-INF下修改专有配置。不会对全局进行影响。

```
mkdir -pv app/{lib,classes,WEB-INF,META-INF}
```

```jsp
<%@ page language="java" %>
<%@ page import="java.util.*" %>
<html>
	 	<head><title>JSP Test Page</title></head>
	 	<body><% out.println("Hellow, world."); %></body>
<html>
```



![image-20200730221506310](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20200730221506310.png)



java程序在运行时，index.jsp文档首先会被转为java文件。最后编译成class文件。这些编译结果会保存在java（tomcat）当中的work子目录。

为了能使每一个类都是全局唯一的，所以每个公司所开发的java类（可能被公共调用的），都以自己公司的域名倒过来写 org.apache.jsp。

```bash
[root@java tomcat]# tree /app/tomcat/work/
/app/tomcat/work/
└── Catalina
    └── localhost
        ├── app
        │   └── org
        │       └── apache
        │           └── jsp 
        │               ├── index_jsp.class #<==最终被编译成java类，最终运行的
        │               └── index_jsp.java  #<==这是servlet文件，纯java编码
        └── ROOT
            └── org
                └── apache
                    └── jsp
                        ├── index_jsp.class
                        └── index_jsp.jav
```

### 3.4 部署

部署(deployment) webapp相关的操作：

deploy:部署，将webapp的源文件放置于目标目录、配置tomcat服务器能够基于context.xml文件中定义的路径来访问此webapp；将其特有类通过class loader装载至tomcat

> **tomcat部署的两种方式**

#### 3.4.1 自动部署autodeploy

#### 3.4.2 手动部署

1. 冷部署：把webapp复制到指定位置，而后才启动tomcat。
2. 热部署：在不停止tomcat的前提下进行的部分。需要通过部署工具来进行。常见的部署工具有<font style="background:#fee904;" size=3>  manager、ant脚本、tcd</font>（tomcat client deployer）等。

3. undeploy：反部署，停止webapp，并从tomcat实例拆除其部分文件和部署名。

stop：停止，不再想用户提供服务。

start：启动处于“停止”状态的webapp

redeploy：重新部署

## 4 tomcat安全规范

### 4.1 telnet管理端口保护

修改默认的8005管理端口不易猜疑的端口（大于1024）或改为-1关闭端口。

修改SHUTDOWN指令为其他字符串

```xml
<Server port='-1' shutdown='SHUTDOWN'>
```

### 4.2 AJP连接端口保护

1、修改默认的ajp 8009端口为不易冲突的大于1024端口。

2、通过iptables规则限制ajp端口访问的权限仅为线上机器。保护此端口的目的在于防止线下测试流量被mod_jk转发至线上tomcat服务器

```xml
<Connector port="8528" protocol="AJP/1.3" />
```

### 4.3 禁用管理端

1. 删除默认的tomcat安装目录 /conf/tomcat-user.xml文件。重启tomcat后将会自动生成新的文件。
2. 删除tomcat目录webapps下默认的所有目录和文件
3. 将tomcat应用根目录配置为tomcat安装目录以外的目录。

### 4.4 降权启动

tomcat启动用户权限必须为非root权限，尽量降低tomcat启动用户的目录访问权限；如果直接对外使用80端口，可通过普通账号启动后配置iptables规则进行转发。避免一旦tomcat服务被入侵，黑客直接获取高级用户权限危害整个server的安全。

### 4.5 版本信息隐藏

1. 修改 `conf/web.xml`，重定向403 404及500等错误指定的错误页面。
2. 可以通过修改应用程序目录下的 `WEB-INF/web.xml` 下的配置进行错误页面的重定向。

```xml
<error-page>
    <error-code>403</error-code>
    <location>/forbidden.jsp</location>
</error-page>

<error-page>
 	<error-code>404</error-code>
 	<location>/notfound.jsp</location>
</error-page>

<error-page>
    <error-code>500</error-code>
 	<location>/systembusy.jsp</location>
</error-page>
```

在配置中对一些常见错误进行重定向，避免当出现错误时tomcat默认显示的错误页面暴露服务器和版本信息，必须确保程序很目录下的错误页面已经存在。

### 4.6 server header重写

在HTTP Connector配置中加入server的配置 `server="webserver" `。当tomcat HTTP端口直接提供web服务时此配置生效，加入此配置，将会替换http响应Server header部分的默认配配置，默认是 `Apache-Coyote/1.1`

### 4.7 访问限制

通过配置限定访问的IP来源。配置信任IP的白名单，拒绝非白名单ip的访间。<font color="#0215cd"  size=2>此配置主要是针对高
保密级别的系统。一般产品线不需要</font>。

```xml
<Context path="" docBasse="/home/work/tomcat" debug="0" reloadable="false" crossContext="true">
      <Valve className="org.apache.catalina.valves.RemoteAddrValue" allow="10.0.0.2,10.0.0.3" deny="*.*.*.*" />
</Context>
```

### 4.8 启动停止脚本权限回收

去除其他用户对tomcat的bin目录下 `shutdown.sh`、`startup.sh`、`catalina.sh` 的可执行权限。防止其他用户有上线的其他权限。

```
chmod -R 744 /app/tomcat/bin/*
```

### 4.9 访问日志规范格式

开启tomcat默认访间日志中的Referer和User-Agent记录

```xml
<Valve className="org.apache.catalina.valvesAccessLogValve"                    
 	directory="logs"
    prefix="localhost_access_log." suffix=".txt"
    pattern="%h %l %u %t %r %s %b %(Referer}i %{User-Agent)i %D" 
 	resolveHosts="false" />
```

### 4.10 文件列表访问控制

`$CATALINA_HOME/conf/web.xml或WEB-INF/web.xml` 文件中default部分 `listings` 的配置必须为 ==`false`==。false为不列出目录列表，true为列出。<font style="background:#fee904;" size=3>默认false</font>。

```xml
 <init-param>
        <param-name>listings</param-name>
        <param-value>false</param-value>
 </init-param>  
```

## 5 tomcat 应用程序

### 5.1 manager app

webapp管理工具

#### 5.1.1 manager配置

首先编辑/app/tomcat/conf/tomcat-users.xml文件，改完后不会立即生效，tomcat在启动时，将读取配置文件到内存中。需要重读授权表或重启。

```
<role rolename="manager-gui"/>
<user username="admin" password="1" roles="manager-gui" />
```

***

**<font color="#0215cd" size=2> <font color="#f8070d" size=2>⚠</font> 请注意，对于Tomcat  manager而言，需要分配您希望访问的功能所需的角色。
</font>**

***

manager-gui：允许访问HTML GUI和状态页面（图形接口）

manager-script：允许访问文本界面和状态页面（命令行接口）

manager-jmx：允许访问JMX代理和状态页面（JMX用来远程监控java程序的监控接口）

manager-status：仅允许访问状态页面（JAVA虚拟机各种状态信息。只读，无法进行管理）



![image-20200730222014902](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20200730222014902.png)



![image-20200730222035238](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20200730222035238.png)



![image-20200730222052169](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20200730222052169.png)



#### 5.1.2 tomcat8 403

```
<Context privileged="true" 
         antiResourceLocking="false"
        docBase="${catalina.home}/webapps/manager">
        <Valve className="org.apache.catalina.valves.RemoteAddrValve" allow="^.*$"  \/>
</Context>
```

### 5.2 host manager

Virtual Hosts管理工具

```
 44   <role rolename="manager-gui"/>
 45   <role rolename="admin-gui" />
 46   <user username="admin" password="1" roles="manager-gui,admin-gui"/>
```



![image-20200730222123975](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20200730222123975.png)



#### 5.2.1 tomcat8 403

```
<Context antiResourceLocking="false" privileged="true" >
      <Valve className="org.apache.catalina.valves.RemoteAddrValve"  
             allow="10.0.0.*" />
      <Manager  
               sessionAttributeValueClassNameFilter="java\.lang\.                                    (?:Boolean|Integer|Long|Number|String)|org\.apache\.catalina\.filters\.CsrfPreventionFilter\$LruCache(?:\$1)?|java\.util\.(?:Linked)?HashMap"/>
</Context>
```

### 5.3 Server Status

manager子组件


## 6 apache反向代理tomcat

### 6.1 lamt

client --> http --> httpd --> reverse_proxy --> http ajp --> tomcat {http connector ajp connector}

> **httpd反代模块**

*  主：proxy_module
*  子：proxy_module_http proxy_module_ajp。

```conf
<VirtualHost *:80>
        DocumentRoot "/app/apache-2.4.28/docs/www"
        ServerName www.test.com
        ServerAlias test.com
        ErrorLog "logs/www.com-error_log"
        CustomLog "logs/www.com-access_log" common
        ProxyVia On
        ProxyRequests Off
        ProxyPreserveHost On
        <Proxy *>
              require all granted
        </Proxy>
        ProxyPass / http://10.0.0.5:8080/
        ProxyPassReverse / http://10.0.0.5:8080/
</VirtualHost>
```

![image-20200730222233836](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20200730222233836.png)



> **ajp**

```
<VirtualHost *:80>
      DocumentRoot "/app/apache-2.4.28/docs/www"
      ServerName www.test.com
      ServerAlias test.com
      ErrorLog "logs/www.com-error_log"
      CustomLog "logs/www.com-access_log" common
      ProxyVia On
      ProxyRequests Off
      ProxyPreserveHost On
      <Proxy *>
           require all granted
      </Proxy>
      ProxyPass /index.php !
      ProxyPass / ajp://10.0.0.5:8009/
      ProxyPassReverse / ajp://10.0.0.5:8009/
</VirtualHost>
```



![image-20200730222353848](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20200730222353848.png)



![image-20200730222322141](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20200730222322141.png)


