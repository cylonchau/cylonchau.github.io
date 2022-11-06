# expect


## 1 什么是Expect

Expent是基于Tcl的相对简单的一个免费的脚本编程工具语言，用来实现自动和交互任务程序进行通信，无需人的手工干预。比如SSH、FTP等，这些程序正常情况都需要手工与它们进行交互，而使用Expect就可以模拟人手工交互的过程，实现自动的和远程的程序交互，从而达到自动化运维的目的。

## 2 Expect程序工作流程

Expect的工作流程可以理解为，spawn启动进程 ==&gt; expect期待关键字 ==&gt; send向进程发送字符 ==&gt; 退出结束。


## 3 安装Expect软件

首先，配置好yum安装源，并且确保机器可以上网，然后执行&lt;font color=&#34;#f8070d&#34; size=3&gt;`yum install expect -y`&lt;/font&gt;即可安装Expect软件。

安装完后查看结果：

```sh
$ rpm -qa|grep expect
expect-5.44.1.15-5.el6_4.x86_64
```

## 4 先看一个Expect小实例

首先准备3台虚拟机：

```sh
192.168.252.60 client
192.168.252.62 client
192.168.252.63 client
192.168.252.64 server
```

再执行下面例子前，我们先手工执行如下命令

```sh
ssh -p52113 root@192.168.252.64 ifconfig
```

执行结果：

```sh
$ ssh -p52113 root@192.168.252.64 ifconfig
root@192.168.252.64&#39;s password: 
```

&gt; &gt; **Expect 例子脚本内容：**

```sh
#!/usr/bin/expect
spawn ssh -p52113 root@192.168.252.62 /sbin/ifconfig eth0
set timeout 60
expect &#34;*password:&#34;
send &#34;111111\n&#34;
expect eof
exit
```

&gt; **执行结果：**问题：

```sh
$ ./a.exp  
spawn ssh -p52113 root@192.168.252.62 /sbin/ifconfig eth0
root@192.168.252.62&#39;s password: 
eth0      Link encap:Ethernet  HWaddr 00:0C:29:C3:48:CB  
          inet addr:192.168.252.62  Bcast:192.168.252.255  Mask:255.255.255.0
          inet6 addr: fe80::20c:29ff:fec3:48cb/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:3027 errors:0 dropped:0 overruns:0 frame:0
          TX packets:1632 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:268416 (262.1 KiB)  TX bytes:175473 (171.3 KiB)
```

&gt; **问题：**

```sh
$ ./a.exp 
./a.exp: line 8: spawn: command not found
couldn&#39;t read file &#34;*password:&#34;: no such file or directory
./a.exp: line 11: send: command not found
couldn&#39;t read file &#34;eof&#34;: no such file or directory
```

原因：
* 含有expect的脚本不能用bash执行，bash无法解析。添加可执行权限后，直接./your_script即可
* 这个代码是expect的代码，不由bash解释。
* spawn可以看作expect脚本的内部函数。要第一行改成 #!/usr/bin/expect

## 5 Expect语法

Expect中的命令是最重要的部分了，命令的使用语法如下：

```sh
命令 [选项] 参数
```


### 5.1 spawn

spawn命令是Expect的初始命令，它用于启动一个进程，之后所有expect操作都在这个进程中进行，如果没有spawn语句，整个expect就无法执行了，spawn使用方法如下：

```sh
spawn ssh root@192.168.252.62
```

在spawn命令后面直接加上要启动的进程、命令等信息，除此之外，spawn还支持其他选项如：
* -open		启动文件进程，具体说明省略。
* -ignore		忽略某些信号，具体说明省略。

### 5.2 expect

使用方法：

```sh
expect  表达式  动作  表达式  动作
```

expect命令用于等候一个相匹配内容的输出，一旦匹配上就执行expect后面的动作或命令。这个命令接受几个特有的参数，用的最多的就是-re，表示使用正则表达式的方式匹配，使用起来就像这样：

```sh
spawn  ssh  root@192.168.252.64
expect  &#34;password:&#34; {send &#34;111111\r&#34;}
```

从上面的例子可以看出，expect是依附与spawn命令的，当执行ssh命令后，expect就匹配命令执行后的关键字：“password：”，如果匹配到关键字就会执行后面包含在“{}”中的send或exp_send动作，匹配的动作可以放在二行，这样就不需要使用“{}”了，就像下面这样，实际完成的功能与上面是一样的：

```sh
spawn  ssh  root@192.168.252.64
expect  &#34;password:&#34; 
send &#34;111111\r&#34;
```

expect命令还有一种用法，他可以在一个expect匹配中多次匹配关键字，并给出处理动作，只需要将关键字放在一个大括号中就可以了，当然还要有exp_continue。

```sh
spawn ssh root@192.168.252.64
expect {
    &#34;yes/no&#34;		{ exp_send &#34;yes\r&#34;; exp_continue }
   &#34;*password:&#34;	{ exp_send &#34;111111\r&#34; }
}
```

### 5.3 exp_send和send

在上面的介绍中，我们已经看到exp_send命令的使用，&lt;font color=&#34;#f8070d&#34; size=3&gt;`exp_send`&lt;/font&gt;命令是&lt;font color=&#34;#f8070d&#34; size=3&gt;`expect`&lt;/font&gt;中的动作命令，它还有一个完成同样工作的同胞：send，exp_send命令可以发送一些特殊符号，我们看到了\r（回车），还有一些其他的比如：\n、\t等等，这些都与TCL中的特殊符号相同。

```sh
spawn ssh root@192.168.252.64
expect {
  &#34;yes/no&#34;		{ exp_send &#34;yes\r&#34;; exp_continue }
  &#34;*password:&#34;	{ exp_send &#34;111111\r&#34; }
}
```

send命令有几个可用参数：

 - -i 制定spawn_id，这个参数用来向不同spawn_id的进程发送命令，是进行多程序控制的关键参数。
 - -s s代表slowly，也就是控制发送的速度，这个参数使用的时候要与expect中的变量send_slow相关联。

### 5.4 exp_continue

这个命令一般用在动作中，它被使用的条件比较苛刻，看看下面的例子：

```sh
#!/usr/bin/expect
spawn ssh root@192.168.252.64 /sbin/ifconfig eth0
set timeout 60
expect {
    -timeout 1
    &#34;yes/no&#34;		{ exp_send &#34;yes\r&#34;; exp_continue }
    &#34;*password:&#34;	{ exp_send &#34;111111\r&#34; }
    timeout 		{ puts  &#34;expect was timeout by oldboy.&#34;;  return}
}
expect eof
exit
```

在这个例子中，可以发现exp_continue命令的使用方法，首先它要处于一个expect命令中，然后它属于一种动作命令，完成的工作就是从头开始遍历，也就是说如果没有这个命令，匹配第一个关键字以后就会继续匹配第二个关键字，但有了这个命令后，匹配第一个关键字以后，第二次匹配仍然从第一个关键字开始。

### 5.5 send_user

send_user命令用来把后面的参数输出到标准输出中去，默认的send、exp_send命令都是将参数输出到程序中去的，用起来就像这样：

```sh
send_user &#34;please input password:&#34;
```

这个语句就可以在标准输出中打印Please input password：字符了。

```sh
#!/usr/bin/expect
if { $argc != 2 } {
send_user &#34;usage: expect scp-expect.exp file host dir\n&#34;
exit
}

#define var
set file [lindex $argv 0]
set host [lindex $argv 1]
set password &#34;111111&#34;
#spawn scp /etc/hosts root@10.0.0.142:/etc/hosts
spawn ssh-copy-id -i $file &#34;-p 52113 root@$host&#34;

expect {
    &#34;yes/no&#34;    {send &#34;yes\r&#34;;exp_continue}
    &#34;*password&#34; {send &#34;$password\r&#34;}
}

expect eof

exit -onexit {
send_user &#34;Oldboy say good bye to you!\n&#34;
}

#script usage
#expect oldboy-6.exp file host dir
#example
#./oldboy-6.exp /etc/hosts 10.0.0.179 /etc/hosts
```

### 5.6 exit

exit命令功能很简单，就是直接退出脚本，但是你可以利用这个命令对脚本做一些扫尾工作，比如下面这样：

```sh
exit -onexit {
    exec  rm  $tmpfile
    send_user &#34;Good bye\n&#34;
}
```

```sh
#!/usr/bin/expect
if { $argc != 2 } {
	send_user &#34;usage: expect scp-expect.exp file host dir\n&#34;
	exit
}

#define var
set file [lindex $argv 0]
set host [lindex $argv 1]
set password &#34;111111&#34;
#spawn scp /etc/hosts root@10.0.0.142:/etc/hosts
spawn ssh-copy-id -i $file &#34;-p 52113 root@$host&#34;

expect {
 	&#34;yes/no&#34;    {send &#34;yes\r&#34;;exp_continue}
 	&#34;*password&#34; {send &#34;$password\r&#34;}
}

expect eof

exit -onexit {
 	send_user &#34;Oldboy say good bye to you!\n&#34;
}

#script usage
#expect oldboy-6.exp file host dir
#example
#./oldboy-6.exp /etc/hosts 10.0.0.179 /etc/hosts
```

## 6 Except变量



expect中有很多有用的变量，它们使用方法与TCL语言中的变量相同，比如：
    set  变量名  变量名   	# 设置变量的方法
    set  $变量名  		  	# 读取变量的方法

```sh
#define var
set file [lindex $argv 0]
set file [lindex $argv 1]
set file [lindex $argv 2]
set password &#34;123456&#34;
```

## 7 Expect关键字

expect中的特殊关键字用于匹配过程，代表某些特殊含义或状态，一般用于expect族命令中不能在外面单独使用，也可以理解为事件，使用上类似于`expect eof {}`


### 7.1 eof

eof (end-of-file)关键字用于匹配 结束符，比如文件的结束符、FTP传输停止等情况，在这个关键字后跟上动作来做进一步的控制，特别是FTP交互操作方面，它的动作很大。用一个例子来看看：

```sh
spawn ftp anonymous@10.11.105.110
expect {
  &#34;password:&#34;  {exp_send &#34;who I’m I&#34;}
   eof {ftp connect close}
}
interact { }
```

### 7.2 timeout

timeout是expect中的一个重要变量，它是一个全局性的时间控制开关，你可以通过为这个变量赋值来规定整个expect操作时间，注意这个变量是服务于expect全局的，它不会纠缠于某一条命令，即使命令没有任何错误，到时见仍然会激活这个变量，但这个时间到达以后除了激活一个开关之外不会做其他的事情，如何处理是脚本编写人员的事情，看看它的实际使用方法：

```sh
set timeout 60
spawn ssh root@192.168.2.1
expect &#34;password:&#34;  {send &#34;word\r&#34;}
expect timeout  { puts &#34;Expect was timeout&#34;; return }
```

上面的处理中，首先将timeout变量设置为60秒，当出现问题的时候程序可能会停止下来，只要到60秒。就会激活下面的timeout动作，这里我们打印一个信息并且停止了脚步的运行--你可以做更多其他的事情，看自己的意思。

在另一种expect格式中，我们还有一种设置timeout变量的方法，看看下面的例子：

```sh
spawn ssh root@192.168.1.1
expect {
    -timeout 60
    -re &#34;password:&#34; {exp_send &#34;word\r&#34;}
    -re &#34;TopsecOS#&#34; {  }
    timeout { puts &#34;Expect was timeout&#34;; return }
    -re &#34;TopsecOS#&#34; {  }
    timeout  { puts &#34;Expecr was timeout&#34;; return }
}
```


## 8 生产场景Expect实战

```sh
$ cat fenfa.exp
#!/usr/bin/expect
if { $argc != 3 } {
         send_user &#34;usage: expect scp-expect.exp file host dir\n&#34;
         exit
}

#define var
set file [lindex $argv 0]
set host [lindex $argv 1]
set dir  [lindex $argv 2]
set password &#34;111111&#34;
spawn scp -P 52113 -p $file root@$host:$dir
#spawn ssh-copy-id -i $file &#34;-p 52113 root@$host&#34;

expect {
    &#34;yes/no&#34;    {send &#34;yes\r&#34;;exp_continue}
    &#34;*password&#34; {send &#34;$password\r&#34;}
}

expect eof

exit -onexit {
    send_user &#34;Oldboy say good bye to you!\n&#34;
}

#script usage
#expect oldboy-6.exp file host dir
#example
#./oldboy-6.exp /etc/hosts 10.0.0.179 /etc/hosts
```

### 实战1：使用expect实现批量分发/etc/hosts文件

```sh
$ cat hosts.sh 
#!/bin/sh
. /etc/init.d/functions
for ip in 192.168.252.60 192.168.252.64  192.168.252.62
do
expect fenfa.exp /etc/hosts/ $ip ~/ &gt;/dev/null 2&gt;&amp;1
if [ $? -eq 0 ];then
    action &#34;$ip&#34; /bin/true
else 
    action &#34;$ip&#34; /bin/false
fi
done
```
如果ip很多写循环列表
```sh
for ip in `cat ip_list`
```
测试结果：
```sh
$ sh hosts.sh   
192.168.252.60                                             [确定]
192.168.252.64                                             [确定]
192.168.252.62                                             [确定]
```

### 实战2：使用expect批量分发SSH密钥文件

首先准备4台机器：

| IP             | hostname-    |
| -------------- | ------------ |
| 192.168.252.60 | lamp_client  |
| 192.168.252.62 | rsync_client |
| 192.168.252.63 | nfs_server   |
| 192.168.252.64 | mysql_client |

&gt; **1 在server端创建公钥与私钥**

```sh
$ ssh-keygen -t dsa
Generating public/private dsa key pair.
Enter file in which to save the key (/root/.ssh/id_dsa): 
Created directory &#39;/root/.ssh&#39;.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_dsa.
Your public key has been saved in /root/.ssh/id_dsa.pub.
The key fingerprint is:
85:c5:f0:88:31:ed:a6:e6:9f:54:61:1e:68:ee:b2:c7 root@server-nfs
The key&#39;s randomart image is:
&#43;--[ DSA 1024]----&#43;
|      o..o.      |
|       &#43;.*.      |
|      ..= *      |
|       oo&#43; o     |
|       oS o      |
|      o. .       |
|     o..o        |
|      .&#43;E.       |
|      .oo        |
&#43;-----------------&#43;
```

&gt; **2 查看密钥**
```sh
[root@test scripts]# ll ~/.ssh/
id_dsa
id_dsa.pub
```

&gt; **3 写批量拷贝脚本**

```sh
#!/usr/bin/expect
if { $argc != 2 } {
    send_user &#34;usage: expect scp-expect.exp file host dir\n&#34;
    exit
}
#define var
set file [lindex $argv 0]
set host [lindex $argv 1]
set password &#34;111111&#34;
#spawn scp /etc/hosts root@10.0.0.142:/etc/hosts
spawn ssh-copy-id -i $file &#34;-p 52113 root@$host&#34;
expect {
     &#34;yes/no&#34;    {send &#34;yes\r&#34;;exp_continue}
     &#34;*password&#34; {send &#34;$password\r&#34;}
}

expect eof

exit -onexit {
     send_user &#34;Oldboy say good bye to you!\n&#34;
}

#script usage
#expect oldboy-6.exp file host dir
#example
#./oldboy-6.exp /etc/hosts 10.0.0.179 /etc/hosts
```

```sh
[root@rsync ~]# ll ~/.ssh/
authorized_keys

[root@nfs scripts]# ll ~/.ssh/
id_dsa
id_dsa.pub
known_hosts

[root@mysql ~]# ll ~/.ssh/
authorized_keys

[root@lamp01 ~]# ll ~/.ssh/
authorized_keys
```

```sh
$ ssh -p52113 root@192.168.252.60 ifconfig
eth0      Link encap:Ethernet  HWaddr 00:0C:29:A7:DB:43  
          inet addr:192.168.252.60  Bcast:192.168.252.255  Mask:255.255.255.0
          inet6 addr: fe80::20c:29ff:fea7:db43/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:1187 errors:0 dropped:0 overruns:0 frame:0
          TX packets:792 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:105294 (102.8 KiB)  TX bytes:73135 (71.4 KiB)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:40 errors:0 dropped:0 overruns:0 frame:0
          TX packets:40 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:3520 (3.4 KiB)  TX bytes:3520 (3.4 KiB)
```

大功告成。

**&lt;font color=&#34;#0215cd&#34; size=2&gt; 特别提示&lt;/font&gt;**：如果是禁止了ROOT远程连接，那么就使用普通用户加sudo的方式结合在expect大量分发。有的同学们问，既然做密钥认证了，为什么还要expect分发呢。在做认证期间，第一次的密钥分发，如果机器数量多，比如1000台，就可以expect分发，比ssh-copy-id方式或http的方式都会好一点。

当然从例一可以看出来，即使不用密钥认证，expect依然可以实现分发数据及文件及批量管理部署服务。
