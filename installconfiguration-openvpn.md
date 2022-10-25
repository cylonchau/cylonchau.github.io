# ch01 cpenvpn install&configuration

[toc]

## 3.1 OpenVPN生产环境需求及环境模拟

| 设备                     | IP                                                           |
| ------------------------ | ------------------------------------------------------------ |
| 笔记本或PC<br>(adsl上网) | 10.0.0.0/24 办公室（DHCP）                                   |
| OpenVPN Server双网卡     | eth0:10.0.0.4/24（外网）<br>eth1:192.168.100.4（内网）       |
| IDC机房内部局域网服务器  | 192.168.100.0/24<br>IDC机房内网服务器无外网IP，又希望ADSL上网笔记本（运维人员），在不同的网络能够直接访问 |
|                          | 实现需求：在远端通过VPN客户端(笔记本)拨号到VPN，然后在笔记本电脑上可以直接访问vpnserver所在局域网内的多个servers，进行维护管理 |
| 环境                     | VPN-Server eth0:10.0.0.4(外网IP)。GW:10.0.0.1, dns:10.0.0.1。<br/><br>eth1:172.0.0.1(内网IP)。GW:不配<br><br/>**提示**：检查是否可以ping通client eth0 IP。<br><br/>App client Server:ethO:172.0.0.2。GW:路由器。<br/><br/>**提示**：检查是否可以ping通VPN-Server eth1 IP。 |

![image-20221024214947488](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20221024214947488.png)

<center>OpenVPN 解决方案图解</center>



## 3.2 部署OpenVPN Server

> **Reference**
>
> [download lzo](http://www.oberhumer.com/opensource/lzo/download/)
>
> [installing-openvpn](https://openvpn.net/community-resources/installing-openvpn/)
>
> [openvpn offical releases](http://build.openvpn.net/downloads/releases/)
>
> [openvpn github](https://github.com/OpenVPN/openvpn/releases)

**安装前准备**

- **openvpn支持的平台：**
  - Linux kernel 2.6+
  - OpenBSD 5.1+
  - Mac OS X Darwin 10.5+
  - FreeBSD 7.4+
  - Windows (WinXP and higher)

- **下载地址：**
  - [openvpn-releases](http://build.openvpn.net/downloads/releases/)
  - [openvpn github](https://github.com/OpenVPN/openvpn/releases)
- **安装openvpn的依赖项：**
  - 0.9.8版或更高版本的OpenSSL库，对于加密是必需的
  - PolarSSL库，是加密的替代版本1.1或更高版本
  - LZO实时压缩库，连接压缩所需

```bash
yum install openssl-devel lzo-devel pam-devel -y
```

```bash
./configure \
--sbindir=/usr/sbin/
--sysconfdir=/etc/openvpn/
--libdir=/usr/lib/
--includedir=/usr/include/
--docdir=/usr/doc
```

使用`rpmbuild`安装：[openvpn.spec](..\..\images\openvpn\openvpn.spec)

## 3.3 配置OpenVPN Server

openvpn的配配置文件在下面目录中

  - `sample/sample-config-files/client.conf`

  - `sample/sample-config-files/server.conf`

```bash
cp sample-config-files/server.conf /etc/openvpn/
```

## 3.3 建立CA证书

easy-rsa是一个基于OpenSSL命令行工具的小型RSA密钥管理包。虽然它主要关注SSL VPN应用程序空间的密钥管理，但它也可用于构建Web证书。它最初被包含在OpenVPN中，但现在是一个单独的项目。

openvpn在2.3-alpha1 -> 2.3-alpha2版本迭代是将easy-rsa分割为单独的子项目。在2.3版本之前easy-rsa包含在openvpn内，在`openvpn-2.2.2/easy-rsa/2.0`目录下。

证书也可以使用openssl进行生成，[easy-rsa](https://github.com/OpenVPN/easy-rsa)可以简化证书生成流程。

### 3.3.1 设置并签署第一个请求

`./easyrsa help [commond]` 查看帮助

`./easyrsa init-pki` 初始化公钥基础设施，（初始化单独的PKI目录）

`./easyrsa build-ca nopass` 创建ca

`./easyrsa gen-req {name} nopass` 创建CSR，{name}只是一个名字，不代表任何。

`./easyrsa import-req pki/{name}.req` 将请求文件`.req`，导入CA

`./easyrsa sign-req {server|client} {name}` 签署请求

`./easyrsa revoke {name}` 吊销证书

`./easyrsa gen-dh`  生成Diffie-Hellman

> Reference
>
> [easyrsa readme](https://github.com/OpenVPN/easy-rsa/blob/master/doc/EasyRSA-Readme.md)

### 3.3.2 EasyRSA的配置源

在easyrsa生成证书时，需要提供证书的配置，来设置证书对应的详情。此时就需要easyrsa获取外部的配置来替换证书默认的参数。

easyrsa有四种方式获得外部配置

- ***命令行选项***

- ***环境变量*** ：覆盖全局选项
- env-vars列表，任何设置/覆盖它的（CLI）以及可能的简洁描述如下所示：
  - `EASYRSA` easyrsa脚本所在的Easy-RSA顶级目录。
  - `EASYRSA_PKI` 保存PKI的文件的目录，默认为`$PWD/pki`。
  - `EASYRSA_DN`：设置为字符串`cn_only`或`org`更改要包含在请求DN中的字段
  - `EASYRSA_REQ_COUNTRY`：DN国家
  - `EASYRSA_REQ_PROVINCE`：DN状态/省
  - `EASYRSA_REQ_CITY`：DN城市/位置
  - `EASYRSA_REQ_ORG`：DN组织
  - `EASYRSA_REQ_EMAIL`：DN电子邮件
  - `EASYRSA_REQ_OU`：DN组织单位
  - `EASYRSA_KEY_SIZE`：设置密钥大小单位
  - `EASYRSA_ALGO`：设置要使用的加密算法：rsa或ec
  - `EASYRSA_CA_EXPIRE`：设置CA到期时间
  - `EASYRSA_CERT_EXPIRE`：设置已颁发证书的到期时间：单位天
  - `EASYRSA_REQ_CN`：默认CN，批量配置时可设置。

- ***vars 文件***：无扩展名的vars文件是为Easy-RSA提供配置的文件，该文件优先级低于环境变量与命令行参数。easyrsa对vars文件的检测顺序为：
  - `--vars` 参数引用的文件
  - 环境变量值`EASYRSA_VARS_FILE`
  - 环境变量设置的pki目录 `EASYRSA_PKI`
  - 执行默认目录 `$PWD/pki`

- ***内部默认值***

> **生成服务端证书和密钥KEY文件**

与上一步类似，其中“Common Name”项需要填写VPN服务器的FQDN，其他均可默认为default值。还会出现：“Sign the certificate? [y/n]”和“1 out of 1 certificate requests certified, commit? [y/n]”。都输入y然后回车，其它可参照如下。

```bash
### 准备生成证书用的CSR相关配置
export EASYRSA_REQ_COUNTRY="HK"
export EASYRSA_REQ_PROVINCE="HK"
export EASYRSA_REQ_CITY="HongKong"
export EASYRSA_REQ_ORG="china mobile"
export EASYRSA_REQ_EMAIL="10086.chinamobile.cn"
#证书有效期
export EASYRSA_CA_EXPIRE=3650
export EASYRSA_CERT_EXPIRE=3650
# 默认cn
export EASYRSA_REQ_CN="csol"
# 批处理模式
export EASYRSA_BATCH="enable"
## 初始化pki
./easyrsa init-pki
## 创建CA
./easyrsa build-ca nopass
## 申请csr
./easyrsa gen-req csol nopass
## 导入csr
./easyrsa import-req ./pki/reqs/csol.req csol
## 签发证书
./easyrsa sign-req server csol
```

> 注意：easyrsa在执行是也是通过openssl进行，需要`openssl-easyrsa.cnf` 与 `x509-types`
>



> Reference
>
> [证书认证](https://blog.dianduidian.com/post/openvpn-server%E6%90%AD%E5%BB%BA%E5%B9%B6%E4%BD%BF%E7%94%A8%E5%AE%A2%E6%88%B7%E7%AB%AF%E8%AF%81%E4%B9%A6%E8%AE%A4%E8%AF%81/)
>
> [openvpn配置文件说明](http://blog.joylau.cn/2020/05/28/OpenVPN-Config/#21-%E6%9C%8D%E5%8A%A1%E7%AB%AF%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6)
>
> [easyrsa签发证书](https://sexywp.com/use-easyrsa-create-self-signed-cert.htm#%E4%B8%8B%E8%BD%BD%E5%AE%89%E8%A3%85_EasyRSA)

## 3.4 加强OpenVPN安全性

`TLS-auth` 参数增加所有 `SSL/TLS` 握手了额外的HMAC签名的完整性验证。任何没有正确HMAC签名的UDP数据包都被丢弃，而无需进一步处理。**TLS-AUTH** 可以防止：

- OpenVPN UDP端口上的DDoS攻击或Flood DDoS 。
- 端口扫描以确定哪些UDP端口处于侦听状态。
- SSL/TLS缓冲区溢出漏洞。
- 切断来自未经授权的机器的`SSL/TLS`握手。

生成随机共享密钥（仅适用于非TLS静态密钥加密模式）

```bash
openvpn --genkey secret ta.key
```

在服务器配置中，添加：

```
tls-auth ta.key 0
```

在客户端配置中，添加：

```
tls-auth ta.key 1
```

> Reference
>
> [openvpn-security](https://openvpn.net/community-resources/hardening-openvpn-security/)

## 3.5 调试OpenVPN服务端

1. 取消服务器上防火墙iptables对Openvpn(默认1194)的拦截。以及允许服务进行转发。

   centos7+ firewalld

   ```bash
   firewall-cmd --add-port=1194/udp  --permanent
   firewall-cmd --permanent --direct --passthrough ipv4 -t nat -A POSTROUTING -s 192.168.100.0/24 -o eth1 -j MASQUERADE
   ```

   iptables

   ```
   iptables -A INPUT -i eth0 -m state --state NEW -p udp --dport 1194 -j ACCEPT
   iptables -t nat -A POSTROUTING -o eth1 -s 192.168.100.0/24 -j MASQUERADE
   ```

2. 开启内核转发功能： 

   ```
   sysctl net.ipv4.ip_forward |grep 1 > /dev/null && \
   echo net.ipv4.ip_forward=1 >> /etc/sysctl.conf 
   ```

3. 启动OpenVPN server

   ```
   openvpn --config server.conf
   ```

4. 默认情况下，openvpn客户端不能访问客户端上具相同子网的IP的主机。

## 3.6 服务端 server.conf参数详解

openvpn默认配置文件在 `sample/sample-config-files/server.conf` 目录下。

```bash
#################################################
# 多客户端服务器的OpenVPN 2.0配置文件示例        #
#                                               #
# 本文件用于多客户端<->单服务器端的              #
# OpenVPN服务器端配置                           #
#                                               #
# OpenVPN也支持单机<->单机的配置                 #
# (在网站上的示例页面更多信息)                   #
#                                               #
# 这个配置可以在Windows或Linux/BSD系统上工作。   #
# Windows的路径名需要加双引号并使用双反斜杠，如： #
# "C:\\Program Files\\OpenVPN\\config\\foo.key" #
#                                               #
# 前面加'#'或';'的是注释                         #
#################################################

# OpenVPN应该监听哪个本地IP地址（可选）
# 如果不设置，默认监听所有IP
;local a.b.c.d

# OpenVPN应该监听哪个端口(TCP/UDP)
# 如果想在同一台计算机上运行多个OpenVPN实例，可以使用不同的端口号来区分它们
# 在防火墙上打开这个端口
port 1194

# 服务器使用TCP还是UDP协议
;proto tcp
proto udp

# 指定OpenVPN创建的通信隧道类型
# "dev tun"将会创建一个路由IP隧道
# "dev tap"将会创建一个以太网隧道
# 如果是以太网桥接模式，并且提前创建了一个名为"tap0"的与以太网接口进行桥接的虚拟接口，则你可以使用"dev tap0"
# 如果想控制VPN的访问策略，必须为TUN/TAP接口创建防火墙规则
# 在非Windows系统中，可以给出明确的单位编号，如"tun0"
# 在Windows中，也可以使用"dev-node"
# 在大多数系统上，除非部分或完全禁用了TUN/TAP接口的防火墙，否则VPN将不起作用。
;dev tap
dev tun

# 如果想配置多个隧道，需要用到网络连接面板中TAP-Win32适配器的名称(如"MyTap")
# 在XP SP2或更高版本的系统中，可能需要有选择地禁用掉针对TAP适配器的防火墙
# 通常情况下，非Windows系统则不需要该指令。
;dev-node MyTap

# 设置SSL/TLS根证书(ca)、证书(cert)和私钥(key)。
# 每个客户端和服务器端都需要它们各自的证书和私钥文件。
# 服务器端和所有的客户端都将使用相同的CA证书文件。
#
# 通过easy-rsa目录下的一系列脚本可以生成所需的证书和私钥。
# 服务器端和每个客户端的证书必须使用唯一的Common Name。
#
# 也可以使用遵循X509标准的任何密钥管理系统来生成证书和私钥。
# OpenVPN也支持使用一个PKCS #12格式的密钥文件(详情查看站点手册页面的"pkcs12"指令)
ca ca.crt
cert server.crt
key server.key  # 该文件应该保密

# 迪菲·赫尔曼参数
# 使用如下命令生成：
#   openssl dhparam -out dh2048.pem 2048
dh dh2048.pem

# 网络拓扑结构
# 应该为子网(通过IP寻址)
# 除非必须支持Windows客户端v2.0.9及更低版本(net30即每个客户端/30)
# 默认为"net30"(不建议)
;topology subnet

# 设置服务器端模式，并提供一个VPN子网，以从中为客户端分配IP地址
# 本例中服务器端自身占用10.8.0.1，其他的将分配给客户端使用
# 每个客户端将能够通过10.8.0.1访问服务器
# 如果使用的是以太网桥接模式，注释掉本行。更多信息请查看官方手册页面。
server 10.8.0.0 255.255.255.0

# 在此文件中维护客户端与虚拟IP地址之间的关联记录
# 如果OpenVPN重启，重新连接的客户端可以被分配到先前分配的虚拟IP地址
ifconfig-pool-persist ipp.txt

# 该指令仅针对以太网桥接模式
# 首先，必须使用操作系统的桥接能力将以太网网卡接口和TAP接口进行桥接
# 然后，需要手动设置桥接接口的IP地址、子网掩码，这里假设为10.8.0.4和255.255.255.0
# 最后，必须指定子网的一个IP范围(例如从10.8.0.50开始，到10.8.0.100结束)，以便于分配给连接的客户端
# 如果不是以太网桥接模式，直接注释掉这行指令即可
;server-bridge 10.8.0.4 255.255.255.0 10.8.0.50 10.8.0.100

# 该指令仅针对使用DHCP代理的以太网桥接模式
# 此时客户端将请求服务器端的DHCP服务器，从而获得分配给它的IP地址和DNS服务器地址
# 在此之前，也需要先将以太网网卡接口和TAP接口进行桥接
# 注意：该指令仅用于OpenVPN客户端(如Windows)，并且该客户端的TAP适配器需要绑定到一个DHCP客户端上
;server-bridge

# 推送路由信息到客户端，以允许客户端能够连接到服务器后的其他私有子网
# 即允许客户端访问VPN服务器可访问的其他局域网
# 记住，这些私有子网还需要将OpenVPN客户端地址池（10.8.0.0/255.255.255.0）路由回到OpenVPN服务器
;push "route 192.168.10.0 255.255.255.0"
;push "route 192.168.20.0 255.255.255.0"

# 要为指定的客户端分配特定的IP地址，或者客户端后的私有子网也要访问VPN
# 可以针对该客户端的配置文件使用ccd子目录
# 请参阅手册页获取更多信息

# 示例1：假设有个Common Name为"Thelonious"的客户端后有一个小型子网也要连接到VPN
# 该子网为192.168.40.128/255.255.255.248
# 首先，去掉下面两行指令的注释：
;client-config-dir ccd
;route 192.168.40.128 255.255.255.248
# 然后创建一个文件ccd/Thelonious，该文件的内容为(没有"#")：
#   iroute 192.168.40.128 255.255.255.248
# 客户端所在的子网就可以访问VPN了
# 注意，这个指令只能在基于路由模式而不是基于桥接模式下才能生效
# 比如，你使用了"dev tun"和"server"指令

# 示例1：假设要给Thelonious分配一个固定的IP地址10.9.0.1
# 首先，去掉下面两行指令的注释：
;client-config-dir ccd
;route 10.9.0.0 255.255.255.252
# 然后在文件ccd/Thelonious中添加如下指令(没有"#")：
#   ifconfig-push 10.9.0.1 10.9.0.2

# 如果想要为不同群组的客户端启用不同的防火墙访问策略，你可以使用如下两种方法：
# (1)运行多个OpenVPN守护进程，每个进程对应一个群组，并为每个进程(群组)启用适当的防火墙规则
# (2)(进阶)创建一个脚本来动态地修改响应于来自不同客户的防火墙规则
# 关于learn-address脚本的更多信息请参考官方手册页面
;learn-address ./script

# 如果启用该行指令，所有客户端的默认网关都将重定向到VPN
# 这将导致诸如web浏览器、DNS查询等所有客户端流量都经过VPN
# (为确保能正常工作，OpenVPN服务器所在计算机可能需要在TUN/TAP接口与以太网之间使用NAT或桥接技术进行连接)
;push "redirect-gateway def1 bypass-dhcp"

# 某些具体的Windows网络设置可以被推送到客户端，例如DNS或WINS服务器地址
# 下列地址来自opendns.com提供的Public DNS服务器
;push "dhcp-option DNS 208.67.222.222"
;push "dhcp-option DNS 208.67.220.220"

# 去掉该行指令的注释将允许不同的客户端之间互相访问
# 默认情况，客户端只能访问服务器
# 为了确保客户端只能看见服务器，还可以在服务器端的TUN/TAP接口上设置适当的防火墙规则
;client-to-client

# 如果多个客户端可能使用相同的证书/私钥文件或Common Name进行连接，那么可以取消该指令的注释
# 建议该指令仅用于测试目的。对于生产环境使用而言，每个客户端都应该拥有自己的证书和私钥
# 如果没有为每个客户端分别生成Common Name唯一的证书/私钥，可以取消该行的注释(不推荐这样做)
;duplicate-cn

# keepalive指令将导致类似于ping命令的消息被来回发送，以便于服务器端和客户端知道对方何时被关闭
# 每10秒钟ping一次，如果120秒内都没有收到对方的回复，则表示远程连接已经关闭
keepalive 10 120

# 出于SSL/TLS之外更多的安全考虑，创建一个"HMAC 防火墙"可以帮助抵御DoS攻击和UDP端口淹没攻击
# 可以使用以下命令来生成：
#   openvpn --genkey --secret ta.key
#
# 服务器和每个客户端都需要拥有该密钥的一个拷贝
# 第二个参数在服务器端应该为'0'，在客户端应该为'1'
tls-auth ta.key 0  # 该文件应该保密

# 选择一个密码加密算法，该配置项也必须复制到每个客户端配置文件中
# 注意，v2.4客户端/服务器将自动以TLS模式协商AES-256-GCM，请参阅手册中的ncp-cipher选项
cipher AES-256-CBC

# 在VPN链接上启用压缩并将选项推送到客户端（仅适用于v2.4 +，对于早期版本，请参阅下文）
;compress lz4-v2
;push "compress lz4-v2"

# 对于与旧客户端兼容的压缩，使用comp-lzo
# 如果在此启用，还必须在客户端配置文件中启用它
;comp-lzo

# 允许并发连接的客户端的最大数量
;max-clients 100

# 初始化后减少OpenVPN守护进程的权限是一个好主意
# 该指令仅限于非Windows系统中使用
;user nobody
;group nobody

# 持久化选项可以尽量避免访问那些在重启之后由于用户权限降低而无法访问的某些资源
persist-key
persist-tun

# 输出一个简短的状态文件，用于显示当前的连接状态，该文件每分钟都会清空并重写一次
status openvpn-status.log

# 默认情况下，日志消息将写入syslog(在Windows系统中，如果以服务方式运行，日志消息将写入OpenVPN安装目录的log文件夹中)
# 可以使用log或者log-append来改变这种默认设置
# "log"方式在每次启动时都会清空之前的日志文件
# "log-append"是在之前的日志内容后进行追加
# 你可以使用两种方式之一(不要同时使用)
;log         openvpn.log
;log-append  openvpn.log

# 为日志文件设置适当的冗余级别(0~9)
# 冗余级别越高，输出的信息越详细
#
# 0 表示静默运行，只记录致命错误
# 4 表示合理的常规用法
# 5和6 可以帮助调试连接错误
# 9 表示极度冗余，输出非常详细的日志信息
verb 3

# 忽略过多的重复信息
# 相同类别的信息只有前20条会输出到日志文件中
;mute 20

# 通知客户端，当服务器重新启动时，可以自动重新连接
# 只能是UDP协议使用，TCP使用的话不能启动服务
explicit-exit-notify 1

# （如果不添加该指令则）默认值3600，也就是一个小时进行一次TSL重新协商
# 这个参数在服务端和客户端设置都有效
# 如果两边都设置了，就按照时间短的设定优先
# 当两边同时设置成0，表示禁用TSL重协商。使用OTP认证需要禁用
reneg-sec 0
```
可用的服务端配置

```conf
local 10.0.0.4
port 1194
proto udp
dev tun
ca ca.crt
cert server.crt
key server.key
dh dh2048.pem
# 分配给客户端的地址池，与dhcp类似
server 192.168.100.0 255.255.255.0
ifconfig-pool-persist ipp.txt

push "route 192.168.100.0 255.255.255.0"
client-to-client
duplicate-cn
keepalive 10 120
comp-lzo
max-clients 100
persist-key
persist-tun
log-append  /var/log/openvpn.log
verb 3
mute 20
explicit-exit-notify 1
reneg-sec 360
tls-auth ta.key 0
```

## 3.7 相关证书文件说明

| 文件名      | 需要使用者               | 目的                      | 默认是否加密 |
| ----------- | ------------------------ | ------------------------- | ------------ |
| ca.crt      | server + all clients     | Root CA certificate       | NO           |
| ca.key      | key signing machine only | Root CA key               | YES          |
| dh{n}.pem   | server only              | Diffie Hellman parameters | NO           |
| server.crt  | server only              | Server Certificate        | NO           |
| server.key  | server only              | Server Key                | YES          |
| client1.crt | client1 only             | Client1 Certificate       | NO           |
| client1.key | client1 only             | Client1 Key               | YES          |

reference [explanation of the relevant files](https://openvpn.net/community-resources/how-to/#openvpn-quickstart)

## 3.8 OpenVPN客户端配置使用

### 3.8.1 下载并安装openvpn客户端

在windows上需要下载与Server版本一致的带有GUI的Windows端。下载地址：[community](https://openvpn.net/community-downloads/)

### 3.8.2 配置并下载客户端证书

**生成客户端证书和key文件**

生成client证书和key文件。若建立多个客户证书，则重复如下步骤即可.只需修改Common Name项oldboy的名称。

在OpenVPN中，这种配置方法是每一个登陆的VPN客户端需要有一个证书，每个证书在同一时刻只能供一个客户端连接（如果有两个机器安装相同证书，同时拨服务器，都能拨上，但是只有第一个拨上的才能连通网络）。所以，如果有多个人，每个人需要建立一份证书。

将ca.crt、redis.crt、redis.key复制到OpenVPN\config。（不同用户使用不同的证书，每个证书包括.crt和.key两个文件。如test.crt和test.key)

在 `sample-config-files/client.conf`的基础上建立客户端配置文件，改名为client.oven，即先在服务器上建立配置文件，然后再下载改名到客户机上。

```bash
client
dev tun
proto udp
remote 10.0.0.4 1194
nobind
resolv-retry infinite
persist-key
persist-tun
mute-replay-warnings
cipher AES-256-CBC
comp-lzo
verb 3
ca ca.crt
cert test.crt
key test.key
tls-auth ta.key 1
```

### 3.8.3 客户端远程连接OpenVPN服务

当配置文件的扩展名为`.ovpn`，配置文件存放至客户端配置的目录中。ca、证书 与配置文件放置同一个文件夹内。每一组为一个用户。

连接成功后如图所示：

![image-20221024215346067](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20221024215346067.png)




