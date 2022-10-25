# 

[TOC]

### 1 配置防火墙，开启FTP服务器需要的端口

CentOS 7.0默认使用的是firewall作为防火墙，这里改为iptables防火墙。

**1、关闭firewall：**

```
systemctl stop firewalld.service 	    #停止firewall
systemctl disable firewalld.service 	#禁止firewall开机启动
```

**2、安装iptables防火墙**

```
yum install iptables-services 					# 安装
vi /etc/sysconfig/iptables 						# 编辑防火墙配置文件
# Firewall configuration written by system-config-firewall
# Manual customization of this file is not recommended.
*filter
:INPUT ACCEPT [0:0]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
-A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
-A INPUT -p icmp -j ACCEPT
-A INPUT -i lo -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 22 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 21 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 10060:10090 -j ACCEPT
-A INPUT -j REJECT --reject-with icmp-host-prohibited
-A FORWARD -j REJECT --reject-with icmp-host-prohibited
COMMIT

:wq! #保存退出

systemctl restart iptables.service #最后重启防火墙使配置生效
systemctl enable iptables.service #设置防火墙开机启动
```

***
> **说明：**

21端口是ftp服务端口；10060到10090是Vsftpd被动模式需要的端口，可自定义一段大于1024的tcp端口。
***

### 2 关闭SELINUX

```
vi /etc/selinux/config
#SELINUX=enforcing 	    # 注释掉
#SELINUXTYPE=targeted   # 注释掉
SELINUX=disabled 		# 增加
:wq! #保存退出
setenforce 0            # 使配置立即生效
```

### 3 安装vsftpd

```
# 安装vsftpd
yum install -y vsftpd 
# 安装vsftpd虚拟用户配置依赖包
yum install -y psmisc net-tools systemd-devel libdb-devel perl-DBI 

systemctl start vsftpd.service # 启动
systemctl enable vsftpd.service # 设置vsftpd开机启动
```

### 4 配置vsftp服务器

**备份默认配置文件**

```
cp /etc/vsftpd/vsftpd.conf /etc/vsftpd/vsftpd.conf-bak
```

**执行以下命令进行设置**

```
sed -i "s/anonymous_enable=YES/anonymous_enable=NO/g" '/etc/vsftpd/vsftpd.conf'
sed -i "s/#anon_upload_enable=YES/anon_upload_enable=NO/g" '/etc/vsftpd/vsftpd.conf'
sed -i "s/#anon_mkdir_write_enable=YES/anon_mkdir_write_enable=YES/g" '/etc/vsftpd/vsftpd.conf'
sed -i "s/#chown_uploads=YES/chown_uploads=NO/g" '/etc/vsftpd/vsftpd.conf'
sed -i "s/#async_abor_enable=YES/async_abor_enable=YES/g" '/etc/vsftpd/vsftpd.conf'
sed -i "s/#ascii_upload_enable=YES/ascii_upload_enable=YES/g" '/etc/vsftpd/vsftpd.conf'
sed -i "s/#ascii_download_enable=YES/ascii_download_enable=YES/g" '/etc/vsftpd/vsftpd.conf'
sed -i "s/#ftpd_banner=Welcome to blah FTP service./ftpd_banner=Welcome to FTP service./g" '/etc/vsftpd/vsftpd.conf'

echo -e "use_localtime=YES\nlisten_port=21\nchroot_local_user=YES\nidle_session_timeout=300
\ndata_connection_timeout=1\nguest_enable=YES\nguest_username=vsftpd
\nuser_config_dir=/etc/vsftpd/vconf\nvirtual_use_local_privs=YES
\npasv_min_port=10060\npasv_max_port=10090
\naccept_timeout=5\nconnect_timeout=1" >> /etc/vsftpd/vsftpd.conf
```

**这是配置好的配置文件**

```
anonymous_enable=NO
local_enable=YES
write_enable=YES
local_umask=022
anon_upload_enable=NO
anon_mkdir_write_enable=YES
dirmessage_enable=YES
xferlog_enable=YES
connect_from_port_20=YES
chown_uploads=NO
xferlog_std_format=YES 	
async_abor_enable=YES
ascii_upload_enable=YES
ascii_download_enable=YES
ftpd_banner=Welcome to FTP service.
listen=NO
listen_ipv6=YES
pam_service_name=vsftpd
userlist_enable=YES
tcp_wrappers=YES
use_localtime=YES
listen_port=21
chroot_local_user=YES
idle_session_timeout=300
data_connection_timeout=1
guest_enable=YES
guest_username=vsftpd
user_config_dir=/etc/vsftpd/vconf
virtual_use_local_privs=NO
pasv_min_port=10060
pasv_max_port=10090
accept_timeout=5
connect_timeout=1
```

### 5 建立虚拟用户名单文件

```
touch /etc/vsftpd/virtualUsers
# 编辑虚拟用户名单文件：（第一行账号，第二行密码，注意：不能使用root做用户名，系统保留）
vi /etc/vsftpd/virtualUsers
public
123456
admin
111111
:wq! 	#保存退出
```

### 6 生成虚拟用户数据文件

```
db_load -T -t hash -f /etc/vsftpd/virtusers /etc/vsftpd/virtualUsers.db
chmod 600 /etc/vsftpd/virtualUsers.db 	#设定PAM验证文件，并指定对虚拟用户数据库文件进行读取
```

### 7 在/etc/pam.d/vsftpd的文件头部加入以下信息

```
# 修改前先备份 
cp /etc/pam.d/vsftpd /etc/pam.d/vsftpdbak
vi /etc/pam.d/vsftpd
auth sufficient /lib64/security/pam_userdb.so db=/etc/vsftpd/virtualUsers
account sufficient /lib64/security/pam_userdb.so db=/etc/vsftpd/virtualUsers
```

***
> 注：

在后面加入无效\
如果系统为32位，上面改为lib，否则配置失败
***

### 8 新建系统用户vsftpd

用户目录为/home/ftp 用户登录终端设为/bin/false(即使之不能登录系统)

```
useradd vsftpd -d /home/ftp -s /bin/false
chown vsftpd:vsftpd /home/ftp -R
chown www:www /home/www-R 	# 如果虚拟用户的宿主用户为www，需要这样设置。
```

### 9 建立虚拟用户个人Vsftp的配置文件

```
# 创建虚拟用户用户的权限配置文件
mkdir /etc/vsftpd/vconf
# 进入该目录
cd /etc/vsftpd/vconf
# 创建public与admin两个用户的配置文件
touch public admin
# 编辑用户admin配置文件，其他的跟这个配置文件类似
vi admin
local_root=/home/ftp/
write_enable=YES
anon_world_readable_only=NO
anon_upload_enable=YES
anon_mkdir_write_enable=YES
anon_other_write_enable=YES
allow_writeable_chroot=YES
```

### 10 最后重启vsftpd服务器

```
systemctl restart vsftpd.service
```

### 11 备注

- guest_username=vsftpd  #指定虚拟用户的宿主用户（就是我们前面新建的用户）
- guest_username=www  #如果ftp目录是指向网站根目录，用来上传网站程序，可以指定虚拟用户的宿主用户为nginx运行账户www，可以避免很多权限设置问题。
- 
#### vsftpd配置文件详解

```
allow_anon_ssl NO
# 只有ss1_enable激活了才可以启用此项。如果设置为YES，匿名用户将容许使用安全的SSL连接服务器。
anon_mkdir_write_enable NO
# 如果设为YES，匿名用户将容许在指定的环境下创建新目录。如果此项要生效，那么配置write_enable必须被激活，并且匿名用户必须在其父目录有写权限。
anon_other_write_enable NO
# 如果设置为YES，匿名用户将被授予较大的写权限，例如删除和改名。一般不建议这么做，除非想完全授权。也可以和cmds_allowed配合来实现控制，这样可以达到文件续传功能。
anon_upload_enable NO
# 如果设为YES，匿名用户就容许在指定的环境下上传文件。如果此项要生效，那么配置write_enable必须激活。并且匿名用户必须在相关目录有写权限。
anon_world_readable_only YES
# 启用的时候，匿名用户只容许下载完全可读的文件，这也就容许了ftp用户拥有对文件的所有权，尤其是在上传的情况下。
anonymous_enable YES
# 控制是否容许匿名用户登录。如果容许，那么“ftp”和“anonymous”都将被视为“anonymous"而容许登录。
ascii_download_enable NO
# 启用时，用户下载时将以ASCII模式传送文件。
ascii_upload_enable NO
# 启用时，用户上传时将以ASCII模式传送文件。
async_abor_enable NO
# 启用时，一个特殊的FTP命令"async ABOR”将容许使用。只有不正常的FTP客户端要使用这一点。而且，这个功能又难于操作，所以，默认是把它关闭了。但是，有些客户端在取消一个传送的时候会被挂死(注：估计是客户端无响应了)，那你只有启用这个功能才能避免这种情况。
background NO
# 启用时，并且VSFTPD是“listen”模式启动的(注：就是standalone模式)，VSFTPD将把监听进程置于后台。但访问VSFTPD时，控制台将立即被返回到SHELL。
check_shell YES
# 注意：这个选项只对非PAM结构的VSFTPD才有效。如果关闭，VSFTPD将不检查/etc/shells以判定本地登录的用户是否有一个可用的SHELL。
chmod_enable YES
# 启用时，将容许使用SITE CHMOD命令。注意，这只能用于本地用户。匿名用户绝不能使用SITE CHMOD。
chown_uploads NO
# 如果启用，所以匿名用户上传的文件的所有者将变成在chown_username里指定的用户。这对管理FTP很有用，也许也对安全有益。
chroot_list_enable NO
# 如果激活，你要提供一个用户列表，表内的用户将在登录后被放在其home目录，锁定在虚根下(注：进入FTP后，PWD一下，可以看到当前目录是"/", 这就是虚根。是FTP的根目录，并非FTP服务器系统的根目录)。如果chroot_local_user设为YES后，其含义会发生一点变化。在这种情况下，这个列表内的用户将不被锁定在虚根下。默认情况下，这个列表文件是/etc/vsftpd.chroot_list, 但你也可以通过修改chroot_list_file来改变默认值。
chroot_local_user NO
# 如果设为YES，本地用户登录后将被(默认地)锁定在虚根下，并被放在他的home目录下。
警告：这个配置项有安全的意味，特别是如果用户有上传权限或者可使用SHELL的话。在你确定的前提下，再启用它。
注意，这种安全暗示并非只存在于VSFTPD，其实是广泛用于所有的希望把用户锁定在虚根下的FTP软件。
connect_from_port_20 NO
# 这用来控制服务器是否使用20端口号来做数据传输。为安全起见，有些客户坚持启用。相反，关闭这一项可以让VSFTPD更加大众化。
deny_email_enable NO
# 如果激活，你要提供一个关于匿名用户的密码E-MAIL表(注：我们都知道，匿名用户是用邮件地址做密码的)以阻止以这些密码登录的匿名用户。默认情况下，这个列表文件是/etc/vsftpd.banner_emails，但你也可以通过设置banned_email_file来改变默认值。
dirlist_enable YES
# 如果设置为NO，所有的列表命令(注：如ls)都将被返回“permission denied”提示。
dirmessage_enable NO
# 如果启用，FTP服务器的用户在首次进入一个新目录的时候将显示一段信息。默认情况下，会在这个目录中查找.message文件，但你也可以通过更改message_file来改变默认值。
download_enable YES
# 如果设为NO，下载请求将返回“permission denied”。
dual_log_enable NO
# 如果启用，两个LOG文件会各自产生，默认的是/var/log/xferlog和/var/log/vsftpd.log。前一个是wu-ftpd格式的LOG，能被通用工具分析。后一个是VSFTPD的专用LOG格式。
force_dot_files NO
# 如果激活，即使客户端没有使用“a”标记，(FTP里)以.开始的文件和目录都会显示在目录资源列表里。但是把"."和".."不会显示。(注：即LINUX下的当前目录和上级目录不会以‘.’或‘..’方式显示)。
force_local_data_ssl YES
# 只有在ssl_enable激活后才能启用。如果启用，所有的非匿名用户将被强迫使用安全的SSL登录以在数据线路上收发数据。
force_local_logins_ssl YES
# 只有在ssl_enable激活后才能启用。如果启用，所有的非匿名用户将被强迫使用安全的SSL登录以发送密码。
guest_enable NO
# 如果启用，所有的非匿名用户登录时将被视为”游客“，其名字将被映射为guest_username里所指定的名字。
hide_ids NO
# 如果启用，目录资源列表里所有用户和组的信息将显示为"ftp".
listen NO
# 如果启用，VSFTPD将以独立模式(standalone)运行，也就是说可以不依赖于inetd或者类似的东东启动。直接运行VSFTPD的可执行文件一次，然后VSFTPD就自己去监听和处理连接请求了。
listen_ipv6 NO
# 类似于listen参数的功能，但有一点不同，启用后VSFTPD会去监听IPV6套接字而不是IPV4的。这个设置和listen的设置互相排斥。
local_enable NO
# 用来控制是否容许本地用户登录。如果启用，/etc/passwd里面的正常用户的账号将被用来登录。
log_ftp_protocol NO
# 启用后，如果xferlog_std_format没有被激活，所有的FTP请求和反馈信息将被纪录。这常用于调试(debugging)。
ls_recurse_enable NO
# 如果启用，"ls -R"将被容许使用。这是为了避免一点点安全风险。因为在一个大的站点内，在目录顶层使用这个命令将消耗大量资源。
no_anon_password NO
# 如果启用，VSFTPD将不会向匿名用户询问密码。匿名用户将直接登录。
no_log_lock NO
# 启用时，VSFTPD在写入LOG文件时将不会把文件锁住。这一项一般不启用。它对一些工作区操作系统问题，如Solaris / Veritas文件系统共存时有用。因为那在试图锁定LOG文件时，有时候看上去象被挂死(无响应)了。(注：这我也不是很理解。所以翻译未必近乎原意。原文如下：It exists to workaround operating system bugs such as the Solaris / Veritas filesystem combination which has been observed to sometimes exhibit hangs trying to lock log files.)
one_process_model NO
# 如果你的LINUX核心是2.4的，那么也许能使用一种不同的安全模式，即一个连接只用一个进程。只是一个小花招，但能提高FTP的性能。请确定需要后再启用它，而且也请确定你的站点是否会有大量的人同时访问。
passwd_chroot_enable (注：这段自己看，无语...)

　　if enabled, along with

　　.BR chroot_local_user

　　, then a chroot() jail location may be specified on a per-user basis. Each

　　user's jail is derived from their home directory string in /etc/passwd. The

　　occurrence of /./ in the home directory string denotes that the jail is at that

　　particular location in the path.

　　默认值：NO
pasv_enable YES
# 如果你不想使用被动方式获得数据连接，请设为NO。
pasv_promiscuous NO
# 如果你想关闭被动模式安全检查(这个安全检查能确保数据连接源于同一个IP地址)的话，设为YES。确定后再启用它(注：原话是：只有你清楚你在做什么时才启用它!)合理的用法是：在一些安全隧道配置环境下，或者更好地支持FXP时(才启用它)。
port_enable YES
# 如果你想关闭以端口方式获得数据连接时，请关闭它。
port_promiscuous NO
# 如果你想关闭端口安全检查(这个检查可以确保对外的(outgoing)数据线路只通向客户端)时，请关闭它。确认后再做!
run_as_launching_user NO
# 如果你想让一个用户能启动VSFTPD的时候，可以设为YES。当ROOT用户不能去启动VSFTPD的时候会很有用(注：应该不是说ROOT用户没有权限启动VSFTPD，而是因为别的，例如安全限制，而不能以ROOT身份直接启动VSFTPD)。强烈警告!!别启用这一项，除非你完全清楚你在做什么(:无语....)!!!随意地启动这一项会导致非常严重的安全问题，特别是VSFTPD没有或者不能使用虚根技术来限制文件访问的时候(甚至VSFTPD是被ROOT启动的)。有一个愚蠢的替代方案是启用deny_file，将其设置为{/*,*..*}等，但其可靠性却不能和虚根相比，也靠不住。
如果启用这一项，其他配置项的限制也会生效。例如，非匿名登录请求，上传文件的所有权的转换，用于连接的20端口和低于1024的监听端口将不会工作。其他一些配置项也可能被影响。
secure_email_list_enable NO
# 如果你想只接受以指定E-MAIL地址登录的匿名用户的话，启用它。这一般用来在不必要用虚拟用户的情况下，以较低的安全限制去访问较低安全级别的资源。如果启用它，匿名用户除非用在email_password_file里指定的E-MAIL做为密码，否则不能登录。这个文件的格式是一个密码一行，而且没有额外的空格(注：whitespace,译为空格，不知道是否正确)。
默认的文件名是：/etc/vsftpd.email_passwords.
session_support NO
# 这将配置是否让VSFTPD去尝试管理登录会话。如果VSFTPD管理会话，它会尝试并更新utmp和wtmp。它也会打开一个pam会话(pam_session)，直到LOGOUT才会关闭它，如果使用PAM进行认证的话。如果你不需要会话纪录，或者想VSFTPD运行更少的进程，或者让它更大众化，你可以关闭它。注：utmp和wtmp只在有PAM的环境下才支持。
setproctitle_enable NO
# 如果启用，VSFTPD将在系统进程列表中显示会话状态信息。换句话说，进程名字将变成VSFTPD会话当前正在执行的动作(等待，下载等等)。为了安全目的，你可以关闭这一项。
ssl_enable NO
# 如果启用，vsftpd将启用openSSL，通过SSL支持安全连接。这个设置用来控制连接(包括登录)和数据线路。同时，你的客户端也要支持SSL才行。注意：小心启用此项.VSFTPD不保证OpenSSL库的安全性。启用此项，你必须确信你安装的OpenSSL库是安全的。
ssl_sslv2 NO
# 要激活ssl_enable才能启用它。如果启用，将容许SSL V2协议的连接。TLS V1连接将是首选。
ssl_sslv3 NO
# 要激活ssl_enable才能启用它。如果启用，将容许SSL V3协议的连接。TLS V1连接将是首选。
ssl_tlsv1 YES
# 要激活ssl_enable才能启用它。如果启用，将容许TLS V1协议的连接。TLS V1连接将是首选。
syslog_enable NO
# 如果启用，系统log将取代vsftpd的log输出到/var/log/vsftpd.log.FTPD的了log工具将不工作。
tcp_wrappers NO
# 如果启用，vsftpd将被tcp_wrappers所支持。进入的(incoming)连接将被tcp_wrappers访问控制所反馈。如果tcp_wrappers设置了VSFTPD_LOAD_CONF环境变量，那么vsftpd将尝试调用这个变量所指定的配置。
ext_userdb_names NO
# 默认情况下，在文件列表中，数字ID将被显示在用户和组的区域。你可以编辑这个参数以使其使用数字ID变成文字。为了保证FTP性能，默认情况下，此项被关闭。
tilde_user_enable NO
# 如果启用，vsftpd将试图解析类似于~chris/pics的路径名(一个"~"(tilde)后面跟着个用户名)。注意，vsftpd有时会一直解析路径名"~"和"~/"(在这里，～被解析成内部登录目录)。～用户路径(～user paths)只有在当前虚根下找到/etc/passwd文件时才被解析。
use_localtime NO
# 如果启用，vsftpd在显示目录资源列表的时候，在显示你的本地时间。而默认的是显示GMT(格林尼治时间)。通过MDTM FTP命令来显示时间的话也会被这个设置所影响。
use_sendfile YES
# 一个内部设定，用来测试在你的平台上使用sendfile()系统呼叫的相关好处(benefit).
userlist_deny YES
# 这个设置在userlist_enable被激活后能被验证。如果你设置为NO，那么只有在userlist_file里明确列出的用户才能登录。如果是被拒绝登录，那么在被询问密码前，用户就将被系统拒绝。
userlist_enable NO
# 如果启用，vsftpd将在userlist_file里读取用户列表。如果用户试图以文件里的用户名登录，那么在被询问用户密码前，他们就将被系统拒绝。这将防止明文密码被传送。参见userlist_deny。
virtual_use_local_privs NO
# 如果启用，虚拟用户将拥有和本地用户一样的权限。默认情况下，虚拟用户就拥有和匿名用户一样的权限，而后者往往有更多的限制(特别是写权限)。
write_enable NO
# 这决定是否容许一些FTP命令去更改文件系统。这些命令是STOR, DELE, RNFR, RNTO, MKD, RMD, APPE 和 SITE。
xferlog_enable NO
# 如果启用，一个log文件将详细纪录上传和下载的信息。默认情况下，这个文件是/var/log/vsftpd.log，但你也可以通过更改vsftpd_log_file来指定其默认位置。
xferlog_std_format NO
# 如果启用，log文件将以标准的xferlog格式写入(wu-ftpd使用的格式)，以便于你用现有的统计分析工具进行分析。但默认的格式具有更好的可读性。默认情况下，log文件是在/var/log/xferlog。但是，你可以通过修改xferlog_file来指定新路径。
数字选项
# 以下是数字配置项。这些项必须设置为非负的整数。为了方便umask设置，容许输入八进制数，那样的话，数字必须以0开始。
accept_timeout 60
# 超时，以秒为单位，设定远程用户以被动方式建立连接时最大尝试建立连接的时间。
anon_max_rate 0　(无限制)
# 对于匿名用户，设定容许的最大传送速率，单位：字节/秒。
anon_umask 077
# 为匿名用户创建的文件设定权限。注意：如果你想输入8进制的值，那么其中的0不同于10进制的0。
connect_timeout 60
# 超时。单位：秒。是设定远程用户必须回应PORT类型数据连接的最大时间。
data_connection_timeout 300
# 超时，单位：秒。设定数据传输延迟的最大时间。时间一到，远程用户将被断开连接。
file_open_mode 0666
# 对于上传的文件设定权限。如果你想被上传的文件可被执行，umask要改成0777。
ftp_data_port 20
# 设定PORT模式下的连接端口(只要connect_from_port_20被激活)。
idle_session_timeout 300
# 超时。单位：秒。设置远程客户端在两次输入FTP命令间的最大时间。时间一到，远程客户将被断开连接。
listen_port 21
# 如果vsftpd处于独立运行模式，这个端口设置将监听的FTP连接请求。
local_max_rate　0(无限制)
#　为本地认证用户设定最大传输速度，单位：字节/秒。
local_umask 077
# 设置本地用户创建的文件的权限。注意：如果你想输入8进制的值，那么其中的0不同于10进制的0。
max_clients 0(无限制)
# 如果vsftpd运行在独立运行模式，这里设置了容许连接的最大客户端数。再后来的用户端将得到一个错误信息。
max_per_ip 0(无限制)
# 如果vsftpd运行在独立运行模式，这里设置了容许一个IP地址的最大接入客户端。如果超过了最大限制，将得到一个错误信息。
pasv_max_port 0(使用任何端口)
# 指定为被动模式数据连接分配的最大端口。可用来指定一个较小的范围以配合防火墙。
pasv_min_port 0(使用任何端口)
# 指定为被动模式数据连接分配的最小端口。可用来指定一个较小的范围以配合防火墙。
trans_chunk_size 0(让vsftpd自行选择)
# 你一般不需要改这个设置。但也可以尝试改为如8192去减小带宽限制的影响。
以下是STRING 配置项
anon_root 无
# 设置一个目录，在匿名用户登录后，vsftpd会尝试进到这个目录下。如果失败则略过。
banned_email_file /etc/vsftpd.banned_emails
# deny_email_enable启动后，匿名用户如果使用这个文件里指定的E-MAIL密码登录将被拒绝。
banner_file 无
# 设置一个文本，在用户登录后显示文本内容。如果你设置了ftpd_banner，ftpd_banner将无效。
chown_username ROOT
# 改变匿名用户上传的文件的所有者。需设定chown_uploads。
chroot_list_file /etc/vsftpd.chroot_list
# 这个项提供了一个本地用户列表，表内的用户登录后将被放在虚根下，并锁定在home目录。这需要chroot_list_enable项被启用。如果chroot_local_user项被启用，这个列表就变成一个不将列表里的用户锁定在虚根下的用户列表了。
cmds_allowed 无
# 以逗号分隔的方式指定可用的FTP命令(post　login. USER, PASS and QUIT 是始终可用的命令)。
其他命令将被屏蔽。这是一个强有力的locking down一个FTP服务器的手段。例如：cmds_allowed=PASV,RETR,QUIT(只允许检索文件)
cmds_allowed=ABOR,APPE,CWD,CDUP,FEAT,LIST,MKD,MDTM,PASS,PASV,PWD,QUIT,RETR,REST,
STOR,STRU,TYPE,USER(支持上传和下载的断点续传等命令)。
详细参考：http://www.nsftools.com/tips/RawFTP.htm
deny_file 无
# 这可以设置一个文件名或者目录名式样以阻止在任何情况下访问它们。并不是隐藏它们，而是拒绝任何试图对它们进行的操作(下载，改变目录层，和其他有影响的操作)。这个设置很简单，而且不会用于严格的访问控制-文件系统权限将优先生效。然而，这个设置对确定的虚拟用户设置很有用。
#  特别是如果一个文件能多个用户名访问的话(可能是通过软连接或者硬连接)，那就要拒绝所有的访问名。
# 建议你为使用文件系统权限设置一些重要的安全策略以获取更高的安全性。如deny_file={*.mp3,*.mov,.private}
dsa_cert_file 无(有一个RSA证书就够了)
# 这个设置为SSL加密连接指定了DSA证书的位置。
email_password_file /etc/vsftpd.email_passwords
# 在设置了secure_email_list_enable后，这个设置可以用来提供一个备用文件。
ftp_username ftp
# 这是用来控制匿名FTP的用户名。这个用户的home目录是匿名FTP区域的根。
ftpd_banner 无(默认的界面会被显示)
# 当一个连接首次接入时将现实一个欢迎界面。
guest_username ftp
# 参见相关设置guest_enable。这个设置设定了游客进入后，其将会被映射的名字。
hide_file 无
# 设置了一个文件名或者目录名列表，这个列表内的资源会被隐藏，不管是否有隐藏属性。但如果用户知道了它的存在，将能够对它进行完全的访问。hide_file里的资源和符合hide_file指定的规则表达式的资源将被隐藏。vsftpd的规则表达式很简单，例如hide_file={*.mp3,.hidden,hide*,h?}
listen_address 无
# 如果vsftpd运行在独立模式下，本地接口的默认监听地址将被这个设置代替。需要提供一个数字化的地址。
listen_address6 无
# 如果vsftpd运行在独立模式下，要为IPV6指定一个监听地址(如果listen_ipv6被启用的话)。需要提供一个IPV6格式的地址。
local_root无
# 设置一个本地(非匿名)用户登录后，vsftpd试图让他进入到的一个目录。如果失败，则略过。
message_file .message
# 当进入一个新目录的时候，会查找这个文件并显示文件里的内容给远程用户。dirmessage_enable需启用。
nopriv_user nobody
# 这是vsftpd做为完全无特权的用户的名字。这是一个专门的用户，比nobody更甚。用户nobody往往用来在一些机器上做一些重要的事情。
pam_service_name ftp
# 设定vsftpd将要用到的PAM服务的名字。
pasv_address 无(地址将取自进来(incoming)的连接的套接字)
# 当使用PASV命令时，vsftpd会用这个地址进行反馈。需要提供一个数字化的IP地址。
rsa_cert_file /usr/share/ssl/certs/vsftpd.pem
# 这个设置指定了SSL加密连接需要的RSA证书的位置。
secure_chroot_dir  /usr/share/empty
# 这个设置指定了一个空目录，这个目录不容许ftp　user写入。在vsftpd不希望文件系统被访问时，目录为安全的虚根所使用。
ssl_ciphers DES-CBC3-SHA
# 这个设置将选择vsftpd为加密的SSL连接所用的SSL密码。详细信息参见ciphers。
user_config_dir 无
# 这个强大的设置容许覆盖一些在手册页中指定的配置项(基于单个用户的)。用法很简单，最好结合范例。如果你把user_config_dir改为/etc/vsftpd_user_conf，那么以chris登录，vsftpd将调用配置文件/etc/vsftpd_user_conf/chris。
user_sub_token 无
# 这个设置将依据一个模板为每个虚拟用户创建home目录。例如，如果真实用户的home目录通过guest_username为/home/virtual/$USER 指定，并且user_sub_token设置为 $USER ，那么虚拟用户fred登录后将锁定在/home/virtual/fred下。
userlist_file /etc/vsftpd.user_list
# 当userlist_enable被激活，系统将去这里调用文件。
vsftpd_log_file /var/log/vsftpd.log
# 只有xferlog_enable被设置，而xferlog_std_format没有被设置时，此项才生效。这是被生成的vsftpd格式的log文件的名字。
# dual_log_enable和这个设置不能同时启用。如果你启用了syslog_enable，那么这个文件不会生成，而只产生一个系统log.
xferlog_file /var/log/xferlog
# 这个设置是设定生成wu-ftpd格式的log的文件名。只有启用了xferlog_enable和xferlog_std_format后才能生效。但不能和dual_log_enable同时启用。
```

#### 数字代码

| 数字代码 | 意义                                   |
| :------: | -------------------------------------- |
|   110    | 重新启动标记应答。                     |
|   120    | 服务在多久时间内ready。                |
|   125    | 数据链路埠开启，准备传送。             |
|   150    | 文件状态正常，开启数据连接端口。       |
|   200    | 命令执行成功。                         |
|   202    | 命令执行失败。                         |
|   211    | 系统状态或是系统求助响应。             |
|   212    | 目录的状态。                           |
|   213    | 文件的状态。                           |
|   214    | 求助的讯息。                           |
|   215    | 名称系统类型。                         |
|   220    | 新的联机服务ready。                    |
|   221    | 服务的控制连接埠关闭，可以注销。       |
|   225    | 数据连结开启，但无传输动作。           |
|   226    | 关闭数据连接端口，请求的文件操作成功。 |
|   227    | 进入passive                            |
|   230    | 使用者登入。                           |
|   250    | 请求的文件操作完成。                   |
|   257    | 显示目前的路径名称。                   |
|   331    | 用户名称正确，需要密码。               |
|   332    | 登入时需要账号信息。                   |
|   350    | 请求的操作需要进一部的命令。           |
|   421    | 无法提供服务，关闭控制连结。           |
|   425    | 无法开启数据链路。                     |
|   426    | 关闭联机，终止传输。                   |
|   450    | 请求的操作未执行。                     |
|   451    | 命令终止：有本地的错误。               |
|   452    | 未执行命令：磁盘空间不足。             |
|   500    | 格式错误，无法识别命令。               |
|   501    | 参数语法错误。                         |
|   502    | 命令执行失败。                         |
|   503    | 命令顺序错误。                         |
|   504    | 命令所接的参数不正确。                 |
|   530    | 未登入。                               |
|   532    | 储存文件需要账户登入。                 |
|   550    | 未执行请求的操作。                     |
|   551    | 请求的命令终止，类型未知。             |
|   552    | 请求的文件终止，储存位溢出。           |
|   553    | 未执行请求的的命令，名称不正确。       |
