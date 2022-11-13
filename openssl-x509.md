# 常用加密算法之数字证书与TLS/SSL


[toc]

## 数字证书

互联网上任意双方之间实现通信时，证书的目的有两种，

- 主机证书，主要实现主机与主机之间进程间通信的。
- 个人证书，主要用作个人通信的，主要用作加密的数据的发送。

主机类证书所拥有的标识主要为`主机名`，主机证书名称一定要与互联网之上访问名称一致，否则此证书为不可信证书。

对于一个安全的通信，应该有以下特征：

- 完整性：消息在传输过程中未被篡改
- 身份验证：确认消息发送者的身份
- 不可否认：消息的发送者无法否认自己发送了信息

显然，数字签名和消息认证码是不符合要求的，这里就需要数字证书来解决其弊端。

数字证书（digital certificate）又称公开密钥认证 PKC（英语：Public key certificate）。是在互联网通信中，方式数字签名的秘钥被篡改，是用来证明公开密钥拥有者的身份。此文件包含了公钥信息、拥有者身份信息（主体）、以及数字证书认证机构（发行者）对这份文件的数字签名，以保证这个文件的整体内容正确无误。

数字证书认证机构 CA (Certificate Authority)：是负责发放和管理数字证书的权威机构。

### 公钥证书的格式标准

`X.509`是密码学中公钥明证PKC的格式标准，所有的证书都符合ITU-T X.509国际标准。X.509证书的结构是用`ASN1` (Abstract Syntax Notation One)进行描述数据结构，并使用`ASN.1`语法进行编码。 

### 证书规范

X.509指的是ITU和ISO联合制定的（RFC5280）里定义的的 `X.509 v3`

前使用最广泛的标准为X.509的 v3版本规范 (RFC5280）, 一般遵从`X.509`格式规范的证书，会有以下的内容：

证书组成结构

| **结构**                   | **说明**                                                     |
| -------------------------- | ------------------------------------------------------------ |
| **版本&amp;nbsp;&amp;nbsp;**       | 现行通用版本是 V3，                                          |
| **序号**                   | 用来识别每一张证书，用来追踪和撤销证书。只要拥有签发者信息和序列号，就可以唯一标识一个证书，最大不能过20个字节；由CA来维护 |
| **主体**                   | 拥有此证书的法人或自然人身份或机器，包括：&lt;br&gt;**国家**（C，Country） &lt;br/&gt;**州/省**（S，State）** &lt;br/&gt;**地域/城市**（L，Location） &lt;br/&gt;**组织/单位**（O，Organization） &lt;br/&gt;**通用名称**（CN，Common Name）：在TLS应用上，此字段一般是域名 |
| **发行者**                 | 以数字签名形式签署此证书的数字证书认证机构                   |
| **有效期(Validity)&amp;nbsp;** | 此证书的有效开始时间，在此前该证书并未生效；此证书的有效结束时间，在此后该证书作废。 |
| **公开密钥用途**           | 指定证书上公钥的用途，例如数字签名、服务器验证、客户端验证等 |
| **公开密钥**               |                                                              |
| **公开密钥指纹**           |                                                              |
| **数字签名**               | 使用信任的CA对内容进行                                       |
| 主体别名                   | 例如一个网站可能会有多个域名（www.jd.com www.360buy.com..）&lt;br&gt;一个组织可能会有多个网站（*.baidu.com tieba.baidu.com），&lt;br/&gt;不同的网域可以一并使用同一张证书，方便实现应用及管理。 |

互联网上任意双方之间实现通信时，证书的目的有两种，

- 主机证书，主要实现主机与主机之间进程间通信的。
- 个人证书，主要用作个人通信的，主要用作加密的数据的发送。

主机类证书所拥有的标识主要为`主机名`，主机证书名称一定要与互联网之上访问名称一致，否则此证书为不可信证书。

### 数字证书文件格式

`X.509`一般推荐使用`PEM` (Privacy Enhanced Mail）格式来存储证书相关的文件。

`.crt` &amp; `.cer`：证书文件后缀名
`.key`: 私钥后缀名
`.csr`：证书请求文件后缀名

### 公钥基础设施（PKI）

公钥基础设施 PKI（Public-Key infrastructure）是为了能够更有效地运用公钥而制定的一系列规范和规格的总称。

#### PKI的组成要素

- 用户：使用PKI的人
  - 注册公钥用户。
  - 使用已注册公钥用户。
- 认证机构：
  - 签证机构：CA Certificate Authority
  - 注册机构：RA
- 仓库
  - 证书吊销列表：CRL；
  - 证书存取库，从签发机构中获得其签发的证书，需要有存取库来提供这些证书。

## SSL/TLS

传输层安全协议，TLS，（Transport Layer Security），其前身为**安全套接层** SSL（Secure Sockets Layer）。SSL3.0为SSL最高版本，3.1 即TLS 1.0

SSL/TLS是世界上应用最广泛的密码通信方法。比如说，当在网上商城中输人信用卡号时，我们的Web浏览器就会使用SSL/TLS进行密码通信。使用SSL/TLS可以对通信对象进行认证，还可以确保通信内容的机密性。

| 协议    | 时间   | 状态           |
| ------- | ------ | -------------- |
| SSL 1.0 | 未公布 | 未公布         |
| SSL 2.0 | 1995年 | 已于2011年弃用 |
| SSL 3.0 | 1996年 | 已于2015年弃用 |
| TLS 1.0 | 1999年 |                |
| TLS 1.1 | 2006年 |                |
| TLS 1.2 | 2008年 |                |
| TLS 1.3 | 2018年 |                |

http协议本身为文本格式，数据发送做文本编码；https协议实现的是二进制格式，数据发送做文本编码。由于ssl的存在，双方在实现通讯时，除了tcp协议三次 握手之外，双方还需做ssl握手会话的过程（认证密钥证书、数据交换等）。ssl会话的建立只能基于IP地址，无法基于主机名识别每一个通信方。一个IP地址在某个应用协议上只能建立一个ssl会话。

TLS采用了分层设计，虽然在TCP/IP协议栈上，SSL增加的半层，其内部实现为多层

- 最底层，基础算法原语的实现，aes，rsa，md5
- 向上一层，选定参数后，符合密码学标准分类的算法的实现
   - aes-128-abc-pkcs7 abc内部块的串联方式 pkcs7 对称加密公钥格式。
- 再向上一层：组合算法实现的半成品。
- 用各种组件拼装而成的种种成品密码学协议/软件  tls ssh。 openssh是利用openssl工具实现的软件程序。

## OpenSSL

OpenSSL是一个开放源代码的软件库，并实现了SSL与TLS协议。OpenSSL可以运行在，MS Windows、Linux、MacOS。OpenSSL已经成为linux基础公共组件，主要由三个开源组件组成：

- `openssl`：多用途的命令行工具，能实现对称加密、非对称加密、单项加密等。各功能分别使用单独子命令来实现。
- `libcrypto`：公共加密库，实现了多种加密算法。
- `libssl`：实现了SSL及TLS的功能库

### OpenSSL命令

OpenSSL命令分为 标准命令`Standard commands`、消息摘要命令`Message Digest commands`、密码命令`Cipher commands`三部分组成。

&gt; - 标准命令。完成某些功能时使用的，如生成随机数、扮演CA签发证书。
&gt;    - enc 对称加密
&gt;    - crl 证书吊销列表
&gt;    - ca 证书签发机构
&gt;    - dgst 消息摘要算法（单项散列函数）
&gt;    - dh 密钥交换算法
&gt;    - req 请求证书生成器
&gt; - 消息摘要命令算法，常用散列函数。
&gt; - 加密命令，所支持的加密算法。

### OpenSSL 命令使用

#### 对称加密 

加密

```
openssl enc -e -des3 -a -salt -in /etc/passwd -out ./passwd
```
解密

```
openssl enc -d -des3 -a -salt -in passwd
```

- `-d` 解密
- `-e` 加密
- `-a` base64编码处理数据

#### 单项加密

openssl中，单项加密即数字签名计算message的摘要信息，为 `openssl dgst...`

单项散列函数通常应用于数字签名于消息认证码（MAC： Message Authentication Code 消息认证码，单项加密的一种延申应用，用于实现再网络通信中保证所传输的数据的完整性。）

```
$ openssl dgst -md5 /var/log/messages
MD5(/var/log/messages)= 4515fae68552c00646ee7e07aac25d1d
```
也可以简写为
```
$ openssl md5 /var/log/messages
MD5(/var/log/messages)= 24483320e6af7ed2cee401aa00c260e6
```
openssl也可以生成用户密码 sslpasswd

```
$ openssl passwd -1 -salt 123456
Password: 
$1$123456$wWKtx7yY/RnLiPN.KaX.z.
```

`$1$123456$wWKtx7yY/RnLiPN.KaX.z.`  这是扩展Unix风格的`crypt(3)` 密码哈希语法。格式为 `$id$salt$encrypted` 前`$id$`符表示加密算法 1标识md5 6标识`SHA-512`，123456是指定的salt,salt为id后的最多16位的对密码加盐。

&gt; **Reference**
&gt; [crypt](https://man7.org/linux/man-pages/man3/crypt.3.html)
&gt; [unix style encrypted](https://unix.stackexchange.com/questions/510990/why-is-the-output-of-openssl-passwd-different-each-time)

#### 使用openssl生成随机数

hex基于16进制编码格式  每个字节4位，4指的是4个字节，一个字节是8位二进制数。4位二进制可以用一个16进制字节标识（一个十六进制对应四个二进制 $4*4=16$） 出现字符 num*2
```bash
$ openssl rand -hex 16
8cd7c7a85e22b4548f6942468028fde4
$ openssl rand -base64 6
c0jDwKIG
```

#### 使用OpenSSL生成自签数字证书

获取证书两种方法：

- 使用证书授权机构
	- 生成签名请求（csr）
	- 将csr发送给CA
	- 从CA处接收签名
- 自签名的证书：自已签发自己的公钥

##### **1. 建立RootCA**

**生成私钥**

| 参数  | 说明                                               |
| :---: | :------------------------------------------------- |
| -new  | 表示生成一个新证书签署请求                         |
| -x509 | 专用于CA生成自签证书，如果不是自签证书则不需要此项 |
| -key  | 生成请求时用到的私钥文件                           |
| -out  | 证书的保存路径                                     |
| -days | 证书的有效期限，单位是day（天），默认是365天       |

```bash
cd /etc/pki/tls/
	
openssl genrsa -out private/cakey.pem 2048
# 路径和名称必须为openssl配置文件中路径的名称
```

##### 2 自签名证书

```bash
openssl req -new \
	-x509 \
	-key private/cakey.pem \
	-out cacert.pem \
	-days 3650 \
	-subj &#34;/C=HK/ST=HK/L=HK/O=chinamobile/OU=SY/CN=CHINA MOBILE&#34;
```

自签名证书是base64编码的，查看证书内容

```bash
openssl x509 -in cacert.pem -noout -text
```

| 参数   | 说明                           |
| ------ | ------------------------------ |
| x509   | 格式                           |
| -noout | 输出时不生成新文件，只显示内容 |
| -text  | 以文本格式输出                 |

**数字证书中主题(Subject)中字段的含义**

- 一般的数字证书产品的主题通常含有如下字段：

| 字段名                                                       | 说明                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 公用名称 &lt;font color=&#34;#f8070d&#34; size=3&gt;CN&lt;/font&gt; (Common Name) | 对于 SSL 证书，&lt;font color=&#34;#f8070d&#34; size=2&gt;一般为网站域名&lt;/font&gt;；而对于代码签名证书则为申请单位名称；而对于客户端证书则为证书申请者的姓名； |
| 单位名称 &lt;font color=&#34;#f8070d&#34; size=3&gt;O&lt;/font&gt;  (Organization Name) | 对于 SSL 证书，一般为网站域名；而对于代码签名证书则为申请单位名称；而对于客户端单位证书则为证书申请者所在单位名称； |
| OU                                                           | 可以理解为公司部门名称                                       |
| 所在城市 &lt;font color=&#34;#f8070d&#34; size=3&gt;L&lt;/font&gt; (Locality)    |                                                              |
| 所在省份 &lt;font color=&#34;#f8070d&#34; size=3&gt;ST&lt;/font&gt;(State/Provice) |                                                              |
| 所在国家 &lt;font color=&#34;#f8070d&#34; size=3&gt;C&lt;/font&gt; (Country）    |                                                              |

可以看到生成了一个Chinamobile的自签的RootCA

![](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/1380340-20201105220215285-526813628.png)

##### 用户或组织向CA申请公钥（证书）

1. 生成私钥

```bash
 mkdir child
 openssl genrsa -out child/child.pem 2048
```

2. 生成证书申请文件 `child.csr`

```bash
openssl req -new \
-key child/child.pem \
-out child/child.csr \
-subj &#34;/C=HK/ST=HK/L=HK/O=chinamobile/OU=SY/CN=*.10086.com&#34;
```

3. 将证书申请文件发送给CA

可以通过任意方式发送申请文件给CA

##### CA颁发证书

```
touch /etc/pki/CA/index.txt
touch /etc/pki/CA/serial # 下一个要颁发的编号 16进制
touch /etc/pki/CA/crlnumber
echo 01 &gt; /etc/pki/CA/serial
```

```
openssl ca -in child/child.csr \ # 签发请求
	-cert cacert.pem \ # CA证书
	-keyfile private/cakey.pem \  # CA私钥
	-out child/a.crt -days 30
```

##### 证书发送给客户端

在应用软件中使用证书

![](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/1380340-20201105221806859-170479134.png)

![](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/1380340-20201105221815048-923869731.png)

#### OpenSSL配置文件注释

`/etc/pki/tls/openssl.cnf ` 定义了管理CA的相关信息。

```cnf
# 语法
# 变量 = 值
# 1. 字符串值最好使用双引号界定，并且其中可以使用&#34;\n&#34;,&#34;\r&#34;,&#34;\t&#34;,&#34;\\&#34;这些转义序列。
# 2. 可以使用 ${变量名} 的形式引用同一字段中的变量，使用 ${字段名::变量名} 的形式引用其它字段中的变量。
# 3. 可以使用 ${ENV::环境变量} 的形式引用操作系统中定义的环境变量，若变量不存在则会导致错误。
# 4. 可以在默认字段定义与操作系统环境变量同名的变量作为默认值来避免环境变量不存在导致的错误。
# 5. 如果在同一字段内有多个相同名称的变量，那么后面的值将覆盖前面的值。
# 6. 可以通过 &#34;.include = 绝对路径&#34; 语法或 OPENSSL_CONF_INCLUDE 环境变量引入其他配置文件(*.cnf)。


# 定义 HOME 的默认值，防止操作系统中不存在 HOME 环境变量。
HOME			= .
# 默认的随机数种子文件，对应 -rand 命令行选项。
RANDFILE		= $ENV::HOME/.rnd

# 扩展对象定义
# 如果没有在 OpenSSL 命令行中定义X.509证书的扩展项，那么就会从下面对扩展对象的定义中获取。
# 定义方法有两种，第一种(反对使用)是存储在外部文件中，也就是这里&#34;oid_file&#34;变量定义的文件。
#oid_file		= $ENV::HOME/.oid
# 第二种是存储在配置文件的一个字段中，也就是这里&#34;oid_section&#34;变量值所指定的字段。
oid_section		= new_oids

# 要将此配置文件用于 &#34;openssl x509&#34; 命令的 &#34;-extfile&#34; 选项，
# 请在此指定包含 X.509v3 扩展的小节名称
#extensions =
# 或者使用一个默认字段中仅包含 X.509v3 扩展的配置文件



[ new_oids ]
# 添加可以被 &#39;ca&#39;, &#39;req&#39;, &#39;ts&#39; 命令使用的扩展对象。格式如下：
# 对象简称 = 对象数字ID
# 下面是一些增强型密钥用法(extendedKeyUsage)的例子
# We can add new OIDs in here for use by &#39;ca&#39;, &#39;req&#39; and &#39;ts&#39;.
# Add a simple OID like this:
# testoid1=1.2.3.4
# Or use config file substitution like this:
# testoid2=${testoid1}.5.6

# Policies used by the TSA examples.
tsa_policy1 = 1.2.3.4.1
tsa_policy2 = 1.2.3.4.5.6
tsa_policy3 = 1.2.3.4.5.7

########################################################################################
###################################  证书签发配置  ######################################
########################################################################################

# openssl 的 ca 命令实现了证书签发的功能，其相关选项的默认值就来自于这里的设置。
# 这个字段只是通过唯一的 default_ca 变量来指定默认CA主配置字段的入口(-name 命令行选项的默认值)


########################################################################################
######################## 默认CA主配置字段，(★)标记表示必需项 #############################
########################################################################################
[ ca ]  #
default_ca	= CA_default		# The default ca section # 默认CA

########################################################################################
######################### 默认CA主配置字段，(★)标记表示必需项 ############################
########################################################################################
[ CA_default ] 
# 保存所有信息的文件夹，这个变量只是为了给后面的变量使用
dir			= /etc/pki/CA

# (★)存放新签发证书的默认目录，证书名就是该证书的系列号，后缀是.pem 。对应 -outdir 命令行选项。
certs		= $dir/certs

# 存放证书吊销列表的
crl_dir		= $dir/crl

# 数据库，颁发了那些证书，以及证书状态、编号。 数据索引。默认不存在的
database	= $dir/index.txt

#unique_subject	= no

#(★)存放新签发证书的默认目录，证书名就是该证书的系列号，后缀是.pem 。对应 -outdir 命令行选项。
new_certs_dir	= $dir/newcerts		# default place for new certs. # 新证书目录

#(★)存放CA自身证书的文件名。对应 -cert 命令行选项。
certificate	= $dir/cacert.pem

#(★)签发证书时使用的序列号文本文件，里面必须包含下一个可用的16进制数字。。需手工创建 
serial		= $dir/serial

# 存放当前(下一个吊销证书编号)CRL编号的文件，需手工创建
crlnumber	= $dir/crlnumber

# 证书吊销列表
crl		= $dir/crl.pem 		

#(★)存放CA自身私钥的文件名。对应 -keyfile 命令行选项。
private_key	= $dir/private/cakey.pem

# 私钥文件路径
RANDFILE	= $dir/private/.rand

# 定义X.509证书扩展项的字段。对应 -extensions 命令行选项。
x509_extensions	= usr_cert


# 当用户需要确认签发证书时可读证书DN域的显示格式。可用值与 x509 指令的 -nameopt 选项相同。
name_opt 	= ca_default		# Subject Name options

# 可用值与 x509 指令的 -certopt 选项相同，不过 no_signame 和 no_sigdump 总被默认设置。
cert_opt 	= ca_default		# Certificate field options

# 是否将证书请求中的扩展项信息加入到证书扩展项中去。取值范围以及解释：
## none: 忽略所有证书请求中的扩展项
## copy: 将证书扩展项中没有的项目复制到证书中
## copyall: 将所有证书请求中的扩展项都复制过去，并且覆盖证书扩展项中原来已经存在的值。
## 此选项的主要用途是允许证书请求提供例如 subjectAltName 之类扩展的值。

# copy_extensions = copy

# 定义生成CRL时需要加入的扩展项字段。对应 -crlexts 命令行选项。
# crl_extensions	= crl_ext

# 新签发的证书默认有效期，以天为单位。依次对应 -days , -startdate , -enddate 命令行选项。
default_days	= 365	# 颁发证书默认有效期

# crl的有效期 30天
default_crl_days= 30

# 默认hash算法
default_md	= sha256
preserve	= no	    # keep passed DN ordering


#(★)定义用于证书请求DN域匹配策略的字段，用于决定CA要求和处理证书请求提供的DN域的各个参数值的规则。
# 策略匹配 ，客户端与ca之间区申请证书信息是否必须匹配，对应 -policy 命令行选项。
policy		= policy_match 

########################################################################################
################################ 为签发的证书设置扩展项 ##################################
########################################################################################

# 变量名称是DN域对象的名称，变量值可以是：
# match: 该变量在证书请求中的值必须与CA证书相应的变量值完全相同，否则拒签。
# supplied: 该变量在证书请求中必须提供(值可以不同)，否则拒签。
# optional: 该变量在证书请求中可以存在也可以不存在(相当于没有要求)。
# 除非preserve=yes或者在ca命令中使用了-preserveDN，否则在签发证书时将删除匹配策略中未提及的对象。

[ policy_match ]
countryName		= match
stateOrProvinceName	= match
organizationName	= match
organizationalUnitName	= optional
commonName		= supplied
emailAddress		= optional

########################################################################################
############## &#34;特征名称&#34;字段包含了用户的标识信息，对应 -subj 命令行选项 ###################
########################################################################################

[ req_distinguished_name ]
countryName = CN  # 必须是两字母国家代码
stateOrProvinceName = # 省份或直辖市
localityName = # 城市
organizationName = # 组织名或公司名
organizationalUnitName = # 部门名称
commonName = # 全限定域名或个人姓名
emailAddress = # Email地址

########################################################################################
################################# 为签发的证书设置扩展项 #################################
########################################################################################

[ extendtsion_name ]
# 基本约束(该证书是否为CA证书)。&#34;CA:FALSE&#34;表示非CA证书(不能签发其他证书的&#34;叶子证书&#34;)。
basicConstraints = CA:FALSE

# 颁发机构密钥标识符(&#34;always&#34;表示始终包含)
authorityKeyIdentifier = keyid:always,issuer

# PKIX工作组推荐将使用者与颁发机构的密钥标识符包含在证书中
subjectKeyIdentifier=hash

# 证书用途，如省略，則可以用于签名外的任何。
# server SSL服务器
# objsign 签名证书
# client 客户端
# email 电子邮件
nsCertType = client

# Netscape Comment（nsComment）是包含注释的字符串扩展名，当在某些浏览器中查看证书时，该注释将显示。
nsComment = &#34;OpenSSL Generated Client Certificate&#34;

# 密钥用法：防否认(nonRepudiation)、数字签名(digitalSignature)、密钥加密(keyEncipherment)。
# 密钥协商(keyAgreement)、数据加密(dataEncipherment)、仅加密(encipherOnly)、仅解密(decipherOnly)
keyUsage = nonRepudiation, digitalSignature, keyEncipherment

# 增强型密钥用法(参见&#34;new_oids&#34;字段)：服务器身份验证、客户端身份验证、时间戳。
extendedKeyUsage = critical,serverAuth, clientAuth, timeStamping

# 使用者备用名称(email, URI, DNS, RID, IP, dirName)
# 例如，DNS常用于实现泛域名证书、IP常用于绑定特定的IP地址、&#34;copy&#34;表示直接复制
subjectAltName = DNS:www.example.com, DNS:*.example.net, IP:192.168.7.1, IP:13::17
```

### OpenSSL扩展密钥用法：生成多域名证书

SAN(Subject Alternative Name) 是 `SSL/TLS` 标准 `x.509` 中定义的一个扩展。使用了SAN字段的 SSL 证书，可以扩展此证书支持的域名，使一个证书可支持多个不同域名的解析。[RFC 5280 4.2.1.6](https://tools.ietf.org/html/rfc5280#section-4.2.1.6)

![](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/1380340-20201112203256164-952424212.png)

![](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/1380340-20201112203458757-2086651260.png)

可以看到面子书与京东的证书的 `Subject Alternative Name` 段中列了大量的域名，使用这种类型的证书能够极大的简化网站证书的管理

#### 使用OpenSSL生成根CA

```bash
cd /etc/pki/tls/
openssl genrsa -out private/cakey.pem 2048

openssl req -new \
	-x509 \
	-key private/cakey.pem \
	-out cacert.pem \
	-days 3650 \
	-subj &#34;/C=HK/ST=HK/L=HK/O=chinamobile/OU=SY/CN=CHINA MOBILE&#34;
```
#### 准备openssl配置文件

```conf
[ v3_req ]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
subjectKeyIdentifier=hash
authorityKeyIdentifier = keyid:always,issuer
nsComment = &#34;OpenSSL Generated Client Certificate&#34;
subjectAltName = DNS:etcd, DNS:hk-etcd, IP:10.0.0.1
nsCertType = client, server
```
可以看到此证书请求文件中会包含 `Subject Alternative Names` 字段，并包含之前在配置文件中填写的域名。

#### 使用 openssl 签署带有 SAN 扩展的证书请求csr

1. 生成私钥

```
mkdir child
openssl genrsa -out child/child.pem 2048
```
2. 生成证书申请文件 `child.csr`

```
openssl req -new \
	-key child/child.pem \
	-out child/child.csr \
	-subj &#34;/C=HK/ST=HK/L=HK/O=chinamobile/OU=SY/CN=hketcd&#34; \
	-config ./openssl.cnf
```
&gt; **单条命令实现方式**
```
openssl req -new \
	-key child/child.pem \
	-subj &#34;/C=HK/ST=HK/L=HK/O=chinamobile/OU=SY/CN=hketcd&#34; \
	-reqexts req_v3 \
	-config &lt;(cat /etc/pki/tls/openssl.cnf \
 			&lt;(printf &#34;[aa]\nsubjectAltName=DNS:etcd, DNS:hk-etcd, IP:10.0.0.1&#34;)) \
	-out child/child.csr
```

3. CA颁发证书

单条命令实现

```
openssl ca \
	-in child/child.csr \
	-cert cacert.pem \
	-keyfile private/cakey.pem \
	-out child/child.crt \
	-days 30 \
	-extensions aa \
	-extfile &lt;(cat /etc/pki/tls/openssl.cnf \
             &lt;(printf &#34;[aa]\nsubjectAltName=DNS:etcd, DNS:hk-etcd, IP:10.0.0.1&#34;))
```

![img](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/1380340-20201112210753782-1479864379.png)
