# docker容器管理




`docker -it -d`
|参数 |说明|
|-|-|
|-i, --interactive |即使不是交互模式也保持stdin打开|
|-d, --detach    |后台运行容器并打印容器ID|
|-t, --tty            |分配一个伪TTY|


> **添加自定义主机映射**

```bash
$ docker run -tid --add-host docker-node:10.0.0.1 centos
61d5824c720f1a32c743a3d0f434e17a7f6860dba1cb5559653a80c064da8073
$ docker exec 61d5824c720f1a32c cat /etc/hosts
ff02::2	ip6-allrouters
10.0.0.1	docker-node
172.17.0.2	61d5824c720f
```
> **添加linux功能**

&nbsp;&nbsp;&nbsp;&nbsp;linux内核特性，提供权限访问控制。如需要特殊权限，不赋权限容器将不能正常运行。
> **将容器pid写入一个文件内**

```bash
$ docker run -itd --cidfile /tmp/pid centos
458d9f4b3cc51a4f0f3abffbc78c643b98a89eef3cdfe263e762ac05d3f5f47d
$ cat /tmp/pid 
458d9f4b3cc51a4f0f3abffbc78c643b98a89eef3cdfe263e762ac05d3f5f47d
```
> **将主机列表添加到容器中**

```bash
--device list 
```
> **设置自定义dns**

```bash
$ docker run -it centos cat /etc/resolv.conf
nameserver 10.0.0.2
nameserver 10.0.0.2
$ docker run -it --dns 8.8.8.8 centos cat /etc/resolv.conf
nameserver 8.8.8.8
```

> **设置容器的环境变量**
```bash
$ docker run -itd -e "TEST=abc" centos
2d3ef722737a0a034151060ef2d8e97b21feee7590917a0e921c21e864d18a47
$ docker attach 2d3ef722737a0
[root@2d3ef722737a /]# echo $TEST
abc
```
> **暴露端口或指定范围的端口号**
```bash
$ docker ps 
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
f7fd3f365512        centos              "/bin/bash"         5 seconds ago       Up 5 seconds        8080/tcp            wonderful_golick
```
> **为容器指定主机名**
```bash
$ docker run -itd -h nginx centos
bdcdf43bf4540d1a9bb794042c6d506c2680be0eab317002697a8049c5667716
$ docker exec bdcdf43bf4540 hostname
nginx
```
> **为容器分配ip**

&nbsp;&nbsp;**创建网络**
```bash
$ docker network create --subnet=10.10.0.0/24 network_test
85d5f3e2cd09e2bd57bc68b56c9341f5b1d4cc1194641715937d8e197cca09f7
```
&nbsp;&nbsp;**查看网络**
```bash
$ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
6af203aae34e        bridge                  bridge              local
b20cfc0864e6        host                     host                  local
85d5f3e2cd09        network_test        bridge               local
c287f5c1181e        none                      null                  local
```
&nbsp;&nbsp;**删除网络**
```bash
$ docker network rm network_test
network_test
```
**指定容器网络**

```bash
$ docker run -idt --net=network_test --ip 10.10.0.3 -h network centos
ad885aebe13fa244748c040121f849385ae5b3b8d243f76cb43695495f20c301
```

##### 查看容器信息
```json
"Networks": {
        "network_test": {
            "IPAMConfig": {
                "IPv4Address": "10.10.0.3"
            },
            "Links": null,
            "Aliases": [
                "ad885aebe13f"
            ],
            "NetworkID": "c1196614dc1eec93c34774f9498ea3254084e1",
            "EndpointID": "a1c491b6155b104ac70194d9b1fa7ab84b6d4",
            "Gateway": "10.10.0.1",
            "IPAddress": "10.10.0.3",
            "IPPrefixLen": 24,
            "IPv6Gateway": "",
            "GlobalIPv6Address": "",
            "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:0a:0a:00:03"
                }
```

> **link建立容器之间的连接**

```bash
$ docker run -tid --name centos centos
8f6e13a26afe60ee2ac5335d419852d70580343f590c2500964f54020e54391c

$ docker exec centos ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.17.0.2  netmask 255.255.0.0  broadcast 172.17.255.255
```

```bash
$ docker run -tid --name link --link centos:nginx.org centos
8f1a542c6058caeeb8b59e02b86e7b9cabc2bc7784d71e355f5d5ca7a6738481

$ docker exec link cat /etc/hosts
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
ff00::0	ip6-mcastprefix
ff02::1	ip6-allnodes
ff02::2	ip6-allrouters
172.17.0.2	*nginx.org* 8f6e13a26afe centos
172.17.0.3	8f1a542c6058
```



log-driver




### docker 容器默认日志保存位置

`/var/lib/docker/containers/container-json.log`
```bash
$ ls -1 /var/lib/docker/containers
8f1a542c6058caeeb8b59e02b86e7b9cabc2bc7784d71e355f5d5ca7a6738481
8f6e13a26afe60ee2ac5335d419852d70580343f590c2500964f54020e54391c

$ ls -1
8f6e13a26afe60ee2ac5335d419852d70580343f590c2500964f54020e54391c-json.log
checkpoints
config.v2.json
hostconfig.json
hostname
hosts
mounts
resolv.conf
resolv.conf.hash|
$ cat 8f6e..1c-json.log 
```
可以在启动时将日志输出到指定位置。

|驱动|介绍|
|-|-|
|**none**    |不输出日志|
|**json-file**| Docker的默认日志记录驱动程序，格式为JSON。 |
|**syslog**   |将日志消息写入syslog|
|**journald** |将日志消息写入journald。 journald守护程序必须在主机上运行。|
|**gelf**     |将日志消息写入Graylog扩展日志格式（GELF）端点，例如Graylog或Logstash。|
|**fluentd**  |将日志消息写入流利（正向输入）。流利的守护程序必须在主机上运行|
|**awslogs**  |将日志消息写入Amazon CloudWatch Logs。|
|**splunk**   |使用HTTP事件收集器将日志消息写入splunk|
|**etwlogs** |将日志消息写为Windows事件跟踪（ETW）事件。仅适用于Windows平台。|
|**gcplogs**  |将日志消息写入Google Cloud Platform（GCP）日志记录。|
|**nats**     |用于Docker的nats NATS日志记录驱动程序。将日志条目发布到NATS服务器。|

### 指定日志驱动测试

```bash
docker run -tid --name nginx --log-driver syslog nginx
```
```bash
curl 172.17.0.4

# 新窗口查看日志可见到容器记录到宿主机的syslog中
$ tail -f /var/log/messages 
Jul 26 01:44:18 docker-node2 systemd: Started Session 8 of user root.
Jul 26 01:44:18 docker-node2 systemd: Starting Session 8 of user root.

Jul 26 01:45:20 docker-node2 5f95cf77c994[2032]: 172.17.0.1 \
- - [25/Jul/2018:17:45:20 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.29.0" "-"
```

> **挂载宿主机的分区到容器**


[Use bind mounts | Docker Documentation](https://docs.docker.com/storage/bind-mounts/#choosing-the--v-or---mount-flag)

> **将容器的端口映射到宿主机上**
```bash
docker run -itd -p 8080:80 centos
```

> **将expose声明的所有端口映射到宿主机的随机端口**
>
> -P
> **容器down掉自动重启**

```bash
docker run -tid --name nginx --restart always nginx

$ docker attach nginx
^C
$ docker ps 
CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS              PORTS               NAMES
07b2196fb6c5        nginx               "nginx -g 'daemon of…"   About a minute ago   Up 4 seconds        80/tcp              nginx
```

> **设置文件描述符大小**
```bash
docker run -itd --name test --ulimit nproc=1024 --ulimit nofile=1024 centos

$ docker attach test

[root@71087eca9986 /]# ulimit -a
open files                      (-n) 1024
max user processes              (-u) 1024
```

### 资源限制

> **例1：限制cpu使用数量**

```bash
 docker run -tid --name cpu2 --cpus=2 centos
```

**使用stress测试cpu使用情况**

测试机器为双核4G硬件资源

***1. 限制两颗CPU***

```bash
[root@1eed573b906a /]# stress -c 13
stress: info: [70] dispatching hogs: 13 cpu, 0 io, 0 vm, 0 hdd
# 使用 docker stats 查看
docker stats cpu2
CONTAINER ID        NAME       CPU %           MEM USAGE / LIMIT      MEM %      
1eed573b906a        cpu2          199.46%           792KiB / 3.686GiB       0.02%
````

在宿主机上top查看

```
%Cpu0  : 99.7 us,  0.3 sy,  0.0 ni,  0.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu1  :100.0 us,  0.0 sy,  0.0 ni,  0.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
```
- 将cpu增加至4核，查看cpu状态
```
%Cpu0  :  0.0 us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu1  : 99.7 us,  0.0 sy,  0.0 ni,  0.3 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu2  :  0.0 us,  0.3 sy,  0.0 ni, 99.7 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu3  : 99.7 us,  0.0 sy,  0.0 ni,  0.3 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
```
***2. 限制1颗CPU***

限制容器只使用1核cpu。观察容器状态。发现容器将使用率均衡在其他核心上。
```
%Cpu0  :  0.0 us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu1  : 30.4 us,  0.0 sy,  0.0 ni, 69.6 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu2  : 19.7 us,  0.0 sy,  0.0 ni, 80.3 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu3  : 49.2 us,  0.0 sy,  0.0 ni, 50.8 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st

%Cpu0  :  0.0 us,  0.3 sy,  0.0 ni, 99.7 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu1  :  0.0 us,  0.3 sy,  0.0 ni, 99.7 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu2  : 49.5 us,  0.0 sy,  0.0 ni, 50.5 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu3  : 49.7 us,  0.3 sy,  0.0 ni, 50.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
```

**结论：对于进程来说是没有 CPU 个数这一概念的，内核只能通过进程消耗的 CPU 时间片来统计出进程占用 CPU 的百分比。这也是我们看到的各种工具中都使用百分比来说明 CPU 使用率的原因。**

官方文档：[Limit a container's resources | Docker Documentation](https://docs.docker.com/config/containers/resource_constraints/)

> **指定固定的 CPU**

```bash
$ docker run -tid --name cpunum --cpuset-cpus="2" stress
31d72f808e8992bcfbfead1d4d7a78e37235e1e42c8b2011b67a15d542829287
$ docker attach cpunum
[root@31d72f808e89 /]# stress -c 4

```
top查看cpu状态，发现只有固定的一个cpu被使用

```bash
%Cpu0  :  0.0 us,  0.3 sy,  0.0 ni, 99.7 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu1  :  0.0 us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu2  :100.0 us,  0.0 sy,  0.0 ni,  0.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu3  :  0.0 us,  0.3 sy,  0.0 ni, 99.7 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
```

限制多个cpunum。
```bash
$ docker run -tid --name cpunum2 --cpuset-cpus="0,3" stress
b4cb513dad71aa9c6cc5578a70253c704dcde59a5b1306943a3967c414e2d9a9
$ docker attach cpunum2
[root@b4cb513dad71 /]# stress -c 4
```
top查看cpu状态
```
%Cpu0  :100.0 us,  0.0 sy,  0.0 ni,  0.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu1  :  0.0 us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu2  :  0.0 us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu3  :100.0 us,  0.0 sy,  0.0 ni,  0.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st

```
> **设置CPU权重**

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;当 CPU 资源充足时，设置 CPU 的权重是没有意义的。只有在容器争用 CPU 资源的情况下， CPU 的权重才能让不同的容器分到不同的 CPU 用量。--cpu-shares 选项用来设置 CPU 权重，它的默认值为 1024。我们可以把它设置为 2 表示很低的权重，但是设置为 0 表示使用默认值 1024。

```bash
docker run -tid --name cpu-test1 --cpuset-cpus="0" --cpu-shares=512 stress
docker run -tid --name cpu-test2 --cpuset-cpus="0" --cpu-shares=0 stress
```

当只有test-1使用cpu资源时的CPU负载

```bash
1717c10e24ae        cpu-test2           0.00%               376KiB / 3.686GiB   0.01%              
8a1e1a944e2b        cpu-test1           99.96%              680KiB / 3.686GiB   0.02%  
```
当test-1与test-2争用资源时的CPU负载
```
1717c10e24ae        cpu-test2           66.91%              668KiB / 3.686GiB   
8a1e1a944e2b        cpu-test1           33.37%              576KiB / 3.686GiB 
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;两个容器分享一个 CPU，所以总量应该是 100%。具体每个容器分得的负载则取决于 --cpu-shares 选项的设置！我们的设置分别是 512 和 1024，则它们分得的比例为 1:2。在本例中如果想让两个容器各占 50%，只要把 --cpu-shares 选项设为相同的值就可以了。

https://www.jb51.net/article/135395.htm

对容器资源的限制 /sys/fs/cgroup/

```bash
docker run -tid --name cpunum2 --cpuset-cpus="0,3" stress
$ cat cpuset/cpuset.cpus
0,3
```
