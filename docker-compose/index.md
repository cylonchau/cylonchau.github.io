# docker compose


Compose是一个定义和管理多容器的工具，使用Python语言编写。使用Compose配置文件描述多个容器应用的架构，比如使用
什么镜像、数据卷、网络、映射端口等；然后一条命令管理所有服务，比如启动、停止、重启等。

## 1、Linux安装Compose

参考网址：[Releases · docker/compose · GitHub](https://github.com/docker/compose/releases)

1. 下载二进制文件

```bash
curl -L \
https://github.com/docker/compose/releases/download/1.22.0/docker-compose-\
`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
```
2. 对二进制文件添加可执行权限

```bash
chmod &#43;x /usr/local/bin/docker-compose
```

3. 测试安装

  docker-compose --version

也可以使用pip工具安装：pip install docker-compose

## 2、使用compose

参考文档：[Docker Compose | Docker Documentation](https://docs.docker.com/compose/)

compose语法详解：[Compose file version 3 reference | Docker Documentation](https://docs.docker.com/compose/compose-file/#reference-and-guidelines)
[Docker compose file 中文参考文档 - CSDN博客](https://blog.csdn.net/guyue35/article/details/53891825)

### 2.1 Compose常用命令选项



| 参数   | 介绍                                                         |
| ------ | ------------------------------------------------------------ |
| build  | 构建或修改Dockerfile后重建服务                               |
| config | 验证和查看compose文件语法&lt;br&gt; `-q`,只验证配置，不输出。 当配置正确时，不输出任何内容，当文件配置错误，输出错误信息。&lt;br&gt;`--services`,打印服务名，一行一个 |
| create |                                                              |
| down   | 停止和删除容器、网络、卷、镜像，这些内容是通过docker-compose up命令创建的.  默认值删除 容器 网络。 |
| logs   | 打印compose service日志输出。                                |
| ps     | 打印compose进程，-q只打印pid                                 |

更多参数参考：[Docker-compose命令详解 - CSDN博客](https://blog.csdn.net/wanghailong041/article/details/52162293)

### 2.2 compose创建tomcat环境

&gt; **Dockerfile**
```bash
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
&gt; **compose**
```bash
version: &#34;3&#34;
services: 
  web:
    build:
      context: .
      dockerfile: &#34;javafile&#34;
    ports:
      - &#34;80:8080&#34;
    image: &#34;tomcat&#34;
    container_name: &#34;tomcat&#34;
```


**验证是否**
```bash
[root@node02 ~]# docker-compose ps
   Name            Command         State            Ports         
------------------------------------------------------------------
root_web_1   bin/catalina.sh run   Up      0.0.0.0:32768-&gt;8080/tcp
[root@node02 ~]# 
```

### 2.3 docker compose语法描述

| 关键词  | 解释        |
| ------- | ----------- |
| version | compose版本 |
