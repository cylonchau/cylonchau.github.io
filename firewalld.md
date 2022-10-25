# firewalld

[toc]

### 1 firewalld的配置文件

以xml格式为主
```bash
/etc/firewalld/
/usr/lib/firewalld/
```
使用时的规则是这样的：当需要一个文件时firewalld会首先到第一个目录中查找，如果找到直接使用，否则会继续到第二个目录中查找。

#### 1.1 配置文件结构
firewalld的配置文件主要有两个文件和三个目录

文件：firewalld.conf、lockdown-whitelist.xml

目录：zone services icmptypes

#### 1.2 firewalld.conf

firewalld的主配置文件，不过非常简单，只有五个配置项

defaultzone：默认使用的zone默认public

minimalmark：标记最小值

cleanUpOnExit：退出当前firewalld后是否清除防火墙规则，默认值为yes;

zones

保存zone配置文件

services

保存service配置文件

icmptypes

保存和icmp类型相关的配置文件

### 2 firewall操作
#### 2.1 查看firewalld状态
```bash
$ firewall-cmd --state
not running

$ systemctl start firewalld

$ firewall-cmd --state
running
```
#### 2.2 不改变状态的条件下重新加载防火墙

```bash
firewall-cmd --reload
```
> **获取支持的区域列表**

```bash
firewall-cmd --get-zone
```
> **获得所有支持的服务**

```bash
$ firewall-cmd --get-services
RH-Satellite-6 amanda-client bacula bacula-client dhcp dhcpv6 dhcpv6-client dns ftp high-availability http https imaps ipp ipp-client ipsec kerberos kpasswd ldap ldaps libvirt libvirt-tls mdns mountd ms-wbt mysql nfs ntp openvpn pmcd pmproxypmwebapi pmwebapis pop3s postgresql proxy-dhcp radius redis rpc-bind samba samba-client smtp ssh telnet tftp tftp-clienttransmission-client vnc-server wbem-https
```

> **获取所有支持的ICMP类型**

```bash
$ firewall-cmd --get-icmptypes
destination-unreachable echo-reply echo-request parameter-problem redirect router-advertisement router-solicitation source-quench time-exceeded
```



```
### 3 firewall-cmd的基础操作
#### 3.1 设置并获得默认区域

​```bash
$ firewall-cmd --set-default-zone=public
success
$ firewall-cmd --get-default-zone
public
```

#### 3.2 在不改变条件下重载防火墙

```bash
$ firewall-cmd --reload
success
```

> **获得所有支持区域**

```bash
$ firewall-cmd --get-services
RH-Satellite-6 amanda-client bacula bacula-client dhcp dhcpv6 dhcpv6-client dns ftp high-availability http https imaps ipp ipp-client ipsec kerberos kpasswd ldap ldaps libvirt libvirt-tls mdns mountd ms-wbt mysql nfs ntp openvpn pmcd pmproxypmwebapi pmwebapis pop3s postgresql proxy-dhcp radius redis rpc-bind samba samba-client smtp ssh telnet tftp tftp-clienttransmission-client vnc-server wbem-https
```
> **列出全部启用区域的信息**
```bash
$ firewall-cmd --list-all-zones
block
  interfaces:
  sources:
  services:
  ports:
  masquerade: no
  forward-ports:
  icmp-blocks:
  rich rules:

dmz
  interfaces:
  sources:
  services: ssh
  ports:
  masquerade: no
  forward-ports:
  icmp-blocks:
  rich rules:
...
...
```
#### 3.3 获得活动区域

```bash
$ firewall-cmd --get-active-zones
public
  interfaces: eth0
```
#### 3.4 禁止掉ssh服务
```bash
firewall-cmd --zone=public --remove-service=ssh
```
#### 3.5 允许某个服务生效

```bash
firewall-cmd --zone=public --add-service=ssh
```
永久添加  `--permanent `

#### 3.6 允许某个服务在一个时间段内生效

```bash
$ firewall-cmd --add-service=ssh --timeout=60
success
$ firewall-cmd --list-all
drop (default, active)
  interfaces: eth0
  sources:
  services: ssh
  ports:
  masquerade: yes
  forward-ports:
  icmp-blocks:
  rich rules:
```
```bash
$ firewall-cmd --list-all
drop (default, active)
  interfaces: eth0
  sources:
  services:
  ports:
  masquerade: yes
  forward-ports:
  icmp-blocks:
  rich rules:
```
#### 3.7 查询某个区域是否开启某项服务
```bash
$ firewall-cmd --zone=public --query-service=ssh
no
$ firewall-cmd --zone=home --query-service=ssh
yes
```
#### 3.8 启动区域端口协议组合

此举将启用端口和协议的组合。端口可以是一个单独的端口 或者是一个端口范围 - 。协议可以是 tcp 或udp。
```bash
$ firewall-cmd --zone=public --add-port=80/tcp
success
```
```bash
$ firewall-cmd --zone=public --add-port=10000-20000/tcp
success
```
```bash
$ firewall-cmd --zone=public --list-all
public
  interfaces:
  sources:
  services: dhcpv6-client
  ports: 80/tcp 10000-20000/tcp
  masquerade: yes
  forward-ports:
  icmp-blocks:
  rich rules:
```
#### 3.9 禁用端口和协议组合
```bash
$ 

success
$ firewall-cmd --zone=public --list-all
public
  interfaces:
  sources:
  services: dhcpv6-client
  ports: 80/tcp
  masquerade: yes
  forward-ports:
  icmp-blocks:
  rich rules:
```
#### 3.10 启动IP伪装功能
此举启用区域的伪装功能。私有网络的地址将被隐藏并映射到一个公有IP。这是地址转换的一种形式，常用于路由。由于内核的限制，伪装功能仅可用于IPv4。
```bash
$ firewall-cmd --add-masquerade --zone=home
success
```
#### 3.11 禁用ip伪装
```bash
firewall-cmd -- -masquerade --zone=home
```
在区域中启用端口转发或映射

语法：
```bash
firewall-cmd [--zone=] --add-forward-port=port=[-]:proto= { :toport=[-] | :toaddr=| :toport=[-]:toaddr=}
```
```bash
$ firewall-cmd --add-forward-port=port=9999:proto=tcp:toaddr=192.168.2.82:toport=80
success
```
http://www.sa-log.com/282.html

http://www.fedora.hk/linux/yumwei/show_15.html

https://yq.aliyun.com/ziliao/94786

启用区域的ICMP阻塞功能
