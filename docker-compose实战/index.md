# docker-compose示例


[toc]

## 使用docker-compose构建LNMP环境。

### 编写Dockerfile

这里采用的是先将nginx php打包为rpm包，然后做成镜像。与直接在容器里编译安装同理的。

&gt; **nginx Dockerfile**

```bash
FROM centos
MAINTAINER lc
RUN yum install -y gcc gcc-c&#43;&#43; openssl-devel make pcre-devel
ADD nginx-1.13.9-1.el7.centos.x86_64.rpm /tmp/
RUN cd /tmp/ &amp;&amp; rpm -ivh nginx-1.13.9-1.el7.centos.x86_64.rpm
ADD nginx.conf /etc/nginx/
EXPOSE 80
CMD [&#34;/usr/sbin/nginx&#34;, &#34;-g&#34;, &#34;daemon off;&#34;]
```
&gt; **php Dockerfile**

```bash
FROM centos
MAINTAINER LC
RUN curl -o /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
RUN curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
RUN yum install zlib-devel \
    libxml2-devel \
    libjpeg-devel \
    libjpeg-turbo-devel \
    freetype-devel \
    libpng-devel gd-devel \
    curl-devel \
    libxslt-devel \
    bzip2-devel \
    gmp-devel \
    readline-devel \
    mcrypt \
    mhash \
    openssl-devel \
    libmcrypt-devel -y
COPY php-7.1.17-1.el7.centos.x86_64.rpm /tmp/
COPY libiconv-1.15-1.el7.centos.x86_64.rpm /tmp/
RUN rpm -ivh /tmp/libiconv-1.15-1.el7.centos.x86_64.rpm
RUN rpm -ivh /tmp/php-7.1.17-1.el7.centos.x86_64.rpm
ADD php-fpm.conf /etc/php/
CMD /usr/sbin/php-fpm &amp;&amp; tail -f /dev/null
EXPOSE 9000
```


### 准备构建容器的配置文件

&gt; **在nginx配置文件中增加解析php的语句**

```nginx
location ~ .*\.(php|php5)$ {
		fastcgi_pass  php.com:9000;$&lt;--这里使用link将php的ip解析过来
		fastcgi_index index.php;
		include fastcgi.conf;
}
```
&gt; **修改php-fpm监听端口为外网通讯的ip**

```bash
$ sed -i &#34;s#listen = 127.0.0.1:900$listen = 0.0.0.0:900$g&#34; php/php-fpm.conf 
$ cat php/php-fpm.conf |grep 9000
listen = 0.0.0.0:9000
```
***
注：此步骤可以在打包RPM时，使用%post在安装后进行修改，免去构建镜像的步骤
***

### 准备RPM包

```bash
$ ls -1 php
libiconv-1.15-1.el7.centos.x86_64.rpm
php-7.1.17-1.el7.centos.x86_64.rpm
phpfile
php-fpm.conf
php.ini

$ ls -1 nginx
nginx-1.13.9-1.el7.centos.x86_64.rpm
nginx.conf
nginxfile
```
### 使用docker-compose一键构建镜像

#### 编写docker-compose.yaml
```yaml
version: &#34;3&#34;
services:
  nginx:
    hostname: nginx
    build:
      context: ./nginx
      dockerfile: nginxfile
    expose:
      - &#34;80&#34;
    ports:
      - &#34;80:80&#34;
    links:
      - mysql
      - php:php.com
    volumes:
      - ./wwwroot:/usr/share/nginx/html/
  php:
    hostname: &#34;php&#34;
    build:
      context: ./php
      dockerfile: phpfile
    ports: 
      - &#34;9000:9000&#34;
    links:
      - mysql:mysql-db
    volumes:
      - ./wwwroot:/usr/share/nginx/html/
  mysql:
    image: mysql:5.6
    hostname: mysql
    ports:
      - &#34;3306:3306&#34;
    environment:
      MYSQL_ROOT_PASSWORD: &#34;Zhang@123&#34;
      MYSQL_USER: &#34;test&#34;
      MYSQL_PASSWORD: &#34;test@123&#34;
```

参考文档： https://hub.docker.com/_/mysql/

#### 检查docker-compose-yaml语法

```bash
$ docker-compose config
services:
  mysql:
    environment:
      MYSQL_PASSWORD: test@123
      MYSQL_ROOT_PASSWORD: Zhang@123
      MYSQL_USER: test
    hostname: mysql
    image: mysql:5.6
    ports:
......
```
***
**&lt;font color=#f8070d; size=2&gt;注：在语法正确时，打印docker-compose.yaml内容，语法出错直接报问题所在位置。&lt;/font&gt;**
***

#### 一键构建所有镜像

```bash
$ docker-compose build
mysql uses an image, skipping

Building php
Step 1/12 : FROM centos
 ---&gt; 5182e96772bf

.....

Step 12/12 : EXPOSE 9000
 ---&gt; Running in 3c5f3c46124c
Removing intermediate container 3c5f3c46124c
 ---&gt; 8791ad17224d
Successfully built 8791ad17224d
Successfully tagged lnmp_php:latest

Building nginx
Step : FROM centos
 ---&gt; 5182e96772bf

.....

Step 8/8 : CMD [&#34;/usr/sbin/nginx&#34;, &#34;-g&#34;, &#34;daemon off;&#34;]
 ---&gt; Using cache
 ---&gt; 8cca531a5c21
Successfully built 8cca531a5c21
Successfully tagged lnmp_nginx:latest
```
#### 管理编排容器

```bash
docker-compose up -d
docker-compose down
docker-compose ps 
```

查看运行结果

## 使用docker-compose一键构建tomcat集群


### 编写Dockerfile


#### nginx Dockerfile
```bash
FROM centos
MAINTAINER lc
RUN yum install -y gcc gcc-c&#43;&#43; openssl-devel make pcre-devel
ADD nginx-1.13.9-1.el7.centos.x86_64.rpm /tmp/
RUN cd /tmp/ &amp;&amp; rpm -ivh nginx-1.13.9-1.el7.centos.x86_64.rpm
ADD nginx.conf /etc/nginx/
EXPOSE 80
CMD [&#34;/usr/sbin/nginx&#34;, &#34;-g&#34;, &#34;daemon off;&#34;]
```

### tomcat Dockerfile
```bash
FROM centos
MAINTAINER lc
ADD apache-tomcat-8.5.29.tar.gz /usr/share/
ADD jdk-8u161-linux-x64.tar.gz /usr/share/
RUN mv /usr/share/apache-tomcat-8.5.29 /usr/share/tomcat
ENV JAVA_HOME=/usr/share/jdk1.8.0_161
WORKDIR /usr/share/tomcat
ENTRYPOINT [&#34;bin/catalina.sh&#34;,&#34;run&#34;]
EXPOSE 8080
```
### 准备配置文件
```nginx
upstream tomcat {
		server tomcat01:8080;
		server tomcat02:8080;
}

server {
		listen 81;
		location / {
		     proxy_pass http://tomcat/;
		     proxy_set_header Host $host;
		     client_max_body_size 10m;
		     proxy_set_header X-Real-IP $remote_addr;
		     proxy_set_header REMOTE-HOST $remote_addr;
		     proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
		 }
}

# 在日志中加入如下配置，来证明访问是负载进行的。

log_format  access  &#39;&#34;$upstream_addr&#34; $remote_addr - $remote_user [$time_local] &#34;$request&#34; &#39;
                    &#39;$status $body_bytes_sent &#34;$http_referer&#34; &#39;
										&#39;&#34;$http_user_agent&#34; &#34;$http_x_forwarded_for&#34;&#39;;
access_log /data/nginx/log/access.log access;

```

```bash
mkdir ./webapps/ROOT
echo jsp-test &gt;webapps/ROOT/index.jsp
```

### 准备构建容器所需的软件

```bash
apache-tomcat-8.5.29.tar.gz
jdk-8u144-linux-x64.tar.gz
tomcatfile
```
### 编写docker-compose文件

```yaml
version: &#34;3&#34;
services:
  nginx:
    hostname: nginx
    build:
      context: ./nginx
      dockerfile: nginxfile
    ports:
      - 80:81
    links:
      - tomcat01:tomcat01
      - tomcat02:tomcat02
    volumes:
      - ./webapps:/usr/share/nginx/html
    depends_on:
      - mysql
      - tomcat01
      - tomcat02
  tomcat01:
    hostname: tomcat01
    build:
      context: ./tomcat
      dockerfile: tomcatfile
    links:
      - mysql:mysql-db
    volumes:
      - ./webapps:/usr/share/tomcat/webapps/
  tomcat02:
    hostname: tomcat02
    build: 
      context: ./tomcat
      dockerfile: tomcatfile
    links: 
      - mysql:mysql-db
    volumes:
      - ./webapps:/usr/share/tomcat/webapps/
  mysql:
    hostname: mysql
    image: mysql:5.5
    ports: 
      - 3306:3306
    environment: 
      MYSQL_ROOT_PASSWORD: 123456
      MYSQL_DATABASE: wordpress
      MYSQL_USER: user
      MYSQL_PASSWORD: 123456
```

### 使用docker-compose一键构建镜像
```bash
$ docker-compose -f docker-compose.yml build 


mysql uses an image, skipping
Building tomcat02
Step : FROM centos
 ---&gt; 5182e96772bf
Step 2/9 : MAINTAINER lc
 ---&gt; Using cache
 ---&gt; 111890e6d42e
Step 3/9 : ADD apache-tomcat-8.5.29.tar.gz /usr/share/
 ---&gt; 46725ed86e2e
Step 4/9 : ADD jdk-8u144-linux-x64.tar.gz /usr/share/
 ---&gt; 431ea9bb9918
Step 5/9 : RUN mv /usr/share/apache-tomcat-8.5.29 /usr/share/tomcat
 ---&gt; Running in 22275e028633
Removing intermediate container 22275e028633
 ---&gt; 0f3919e97b50
Step 6/9 : ENV JAVA_HOME=/usr/share/jdk1.8.0_144
 ---&gt; Running in 60d616e0b9fc
Removing intermediate container 60d616e0b9fc
 ---&gt; 5255488c51ab
Step 7/9 : WORKDIR /usr/share/tomcat
 ---&gt; Running in 6b04d8f5524b
Removing intermediate container 6b04d8f5524b
 ---&gt; db88a81ec00a
Step 8/9 : ENTRYPOINT [&#34;bin/catalina.sh&#34;,&#34;run&#34;]
 ---&gt; Running in 189b6ee0ff4e
Removing intermediate container 189b6ee0ff4e
 ---&gt; 9b444829ed55
Step 9/9 : EXPOSE 8080
 ---&gt; Running in f2115bf03afb
Removing intermediate container f2115bf03afb
 ---&gt; 31c01ca93305
Successfully built 31c01ca93305
Successfully tagged lnmp_tomcat02:latest
Building tomcat01
Step : FROM centos
 ---&gt; 5182e96772bf
Step 2/9 : MAINTAINER lc
 ---&gt; Using cache
 ---&gt; 111890e6d42e
Step 3/9 : ADD apache-tomcat-8.5.29.tar.gz /usr/share/
 ---&gt; Using cache
 ---&gt; 46725ed86e2e
Step 4/9 : ADD jdk-8u144-linux-x64.tar.gz /usr/share/
 ---&gt; Using cache
 ---&gt; 431ea9bb9918
Step 5/9 : RUN mv /usr/share/apache-tomcat-8.5.29 /usr/share/tomcat
 ---&gt; Using cache
 ---&gt; 0f3919e97b50
Step 6/9 : ENV JAVA_HOME=/usr/share/jdk1.8.0_144
 ---&gt; Using cache
 ---&gt; 5255488c51ab
Step 7/9 : WORKDIR /usr/share/tomcat
 ---&gt; Using cache
 ---&gt; db88a81ec00a
Step 8/9 : ENTRYPOINT [&#34;bin/catalina.sh&#34;,&#34;run&#34;]
 ---&gt; Using cache
 ---&gt; 9b444829ed55
Step 9/9 : EXPOSE 8080
 ---&gt; Using cache
 ---&gt; 31c01ca93305
Successfully built 31c01ca93305
Successfully tagged lnmp_tomcat01:latest
Building nginx
Step : FROM centos
 ---&gt; 5182e96772bf
Step 2/8 : MAINTAINER lc
 ---&gt; Using cache
 ---&gt; 111890e6d42e
Step 3/8 : RUN yum install -y gcc gcc-c&#43;&#43; openssl-devel make pcre-devel
 ---&gt; Using cache
 ---&gt; 36090d81ef5e
Step 4/8 : ADD nginx-1.13.9-1.el7.centos.x86_64.rpm /tmp/
 ---&gt; Using cache
 ---&gt; cecd606c7619
Step 5/8 : RUN cd /tmp/ &amp;&amp; rpm -ivh nginx-1.13.9-1.el7.centos.x86_64.rpm
 ---&gt; Using cache
 ---&gt; 8c7b331fc175
Step 6/8 : ADD nginx.conf /etc/nginx/
 ---&gt; 84940ae84e06
Step 7/8 : EXPOSE 80
 ---&gt; Running in 94ac22711605
Removing intermediate container 94ac22711605
 ---&gt; ef10799c0844
Step 8/8 : CMD [&#34;/usr/sbin/nginx&#34;, &#34;-g&#34;, &#34;daemon off;&#34;]
 ---&gt; Running in 039ea9de17f0
Removing intermediate container 039ea9de17f0
 ---&gt; d489d87223f8
Successfully built d489d87223f8
Successfully tagged lnmp_nginx:latest
```

测试访问结果

![image](../../images/docker-compose%E5%AE%9E%E6%88%98/e1fcd3f3.png)

查看nginx访问日志，发现是负载到每一台tomcat上的。

```bash
&#34;172.24.0.5:8080&#34; 10.0.0.1 - - [12/Aug/2018:17:52:39 &#43;0000] &#34;GET /index.jsp HTTP/1.1&#34; 200 9 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/65.0.3325.Safari/537.36&#34; &#34;-&#34;
&#34;172.24.0.4:8080&#34; 10.0.0.1 - - [12/Aug/2018:17:52:40 &#43;0000] &#34;GET /index.jsp HTTP/1.1&#34; 200 9 &#34;-&#34; &#34;Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/65.0.3325.Safari/537.36&#34; &#34;-&#34;
```
