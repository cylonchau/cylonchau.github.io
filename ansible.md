# ansible介绍


## 运维工具

**OS Provisioning: PXE, Cobbler(repository,distritution, profile)**

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;PXE: dhcp, tftp, (http, ftp)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;dnsusq: dhcp, dns

**OS Config:**

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;puppet, saltstack, func

**Task Excute:**

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;fabric, func, saltstack

**Deployment:**

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;fabric

管理主机，要想管理被管理节点，二者必须有安全管理通道。puppet、saltstack在管理被管节点时，每一个被管节点必须运行puppet agent，管理端进程与每一个被管节点的agent进程进行通信，通信时使用的HTTP协议。此种方式必须在被管节点安装应用程序的agent，远程接收到指令，并在本地负责执行相应的任务。

根据远程管理时，是不是在每一个被管主机上安装agent端，分为两种puppet、func、saltstack；与无需agent端，ansible、fabric。依赖被管节点的ssh服务。而管理端需要知道对方主机上的账号密码。

## ansible的组成

### ansible核心

host invenory ：为了管控每个被管主机，每个主机在本地需要注册。用来定义由ansible远程配置管理的主机，每个主机的IP地址、掩码、SSH监听的地址、端口号、账号密码等。

core modules：ansible执行任何特定管理任务，都是不由ansible自身玩成的，而是通过模块完成的。

custom Modules：使用任何编程语言来编写模块。

playbooks：将主机要完成的多个任务，事先定义在文件中可以多次调用。

## 1 ansible的特性

*   基于Python语言实现，由Paramiko, PyYAML和jiniia2三个关键模块
*   部署简单，agentless
*   默认使用SSH协议
*   主从模式：
    *  master:ansible, ssh client
    *  slave: ssh server
*  支持自定文模块：支持各种编程语言，支持Playbook，基于“模块”完成各种“任务”。

***
**安装依赖于epel源**
***

> **配置文件：**

```      
/etc/ansible/ansible.cfg
Invertory:/etc/ansible/hosts
```

## 2 ansible命令应用基础

**语法:**    *`ansible <host-pattern> [-f forks] [-m module_name] [-a args]`*

| 参数           | 说明               |
| -------------- | ------------------ |
| -f forks       | 启助的并发线程数。 |
| -m module_name | 要使用的模块。     |
| -a args        | 模块特有的参数     |

####  查看模块

```
ansible-doc -l
command           	Executes a command on a remote node                            
composer         	Dependency Manager for PHP                                     
consul          	Add, modify & delete services within a consul cluster.         
consul_acl      	Manipulate Consul ACL keys and rules                           
consul_kv       	Manipulate entries in the key/value store of a consul cluster. 
consul_session    	manipulate consul sessions                                     
copy               	Copies files to remote locations    
```

```
[root@java ~]# ansible-doc -s command
- name: Executes a command on a remote node
  command:
    chdir:# Change into this directory before running the command.
     creates: A filename or (since 2.0) glob pattern, when it already exists, this step will *not* be run.
```

### 2.1 常见模块

*`command`*：<font color=#0215cd size=3>命令模块，默认模块，用于在远程执行命令。</font>

```
[root@java ~]# ansible 10.0.0.12 -m command -a 'date'
10.0.0.12 | SUCCESS | rc=0 >>
Wed May  2 14:40:57 CST 2018

[root@java ~]# ansible 10.0.0.12 -a 'date'
10.0.0.12 | SUCCESS | rc=0 >>
Wed May  2 14:41:03 CST 2018
```

> ***cron***
>> *state：表示任务是添加还是移除。present表示必须提供的，absent表示必须不能提供的
present：安装、absent移除。*可以不写，默认都是*。state默认 present*

```
ansible websrvs cron -a 'minute="*/10" job="/bin/echo hellow' name-"test cron job"'
```

> **`user`：指明创建用户的名字**

```
ansible all -m user -a 'name=user1'
```

```
ansible all -m group -a 'name=mysql gid=2000 system=yes'
ansible all -m user -a 'user=mysql group=mysql system=yes uid=3306'
```

> ***`copy`***
>> *src本地原路径，可以使用相对和绝对路径
dest远程目标路径，必须使用绝对路径*

```
ansible all -m copy -a 'src=/etc/hosts dest=/tmp/h.ansible' owner='root' mode='644'
```

***<font color=#0215cd size=3>content：取代src，表示直接用此处指定的信息生成为目标文件内容</font>***

```
ansible all -m copy -a 'content="test ansible" dest=/tmp/h.ansible' 

[root@redis ~]# ll /tmp/
total 56
-rw-r--r--. 1 root root   12 May  2 14:50 h.ansible
```

> ***`file`***
>
> > *设置文件属性，path指定文件路径，可以用name或dest替代。*

**创建文件的符号链接**

```
ansible all -m file -a 'path=/tmp/h src=/tmp/h.ansible state=link'

[root@iZ8o31djppqzqeZ conf]#  ll /tmp/
lrwxrwxrwx 1 root root   14 May  2 23:08 h -> /tmp/h.ansible
-rw-r--r-- 1 root root   12 May  2 23:05 h.ansible
```

```
ansible all -m file -a 'name=/tmp/h.123 src=/tmp/h.ansible state=link'

[root@redis ~]# ll /tmp/
total 56
lrwxrwxrwx. 1 root root   14 May  2 14:53 h -> /tmp/h.ansible
lrwxrwxrwx. 1 root root   14 May  2 15:00 h.123 -> /tmp/h.ansible
```

**删除文件夹**

```
ansible all -m file -a 'path=/tmp/ state=absent'
```

**创建文件夹**


```
ansible all -m file -a 'path=/tmp/test-create state=directory'
```

<font color=#0215cd size=2>state=file如果文件不存在，将失败，请使用copy或templates创建</font>

```
ansible-doc -s file
```

**修改文件权限**

```
ansible test -m file -a 'owner=mysql group=mysql mode=000 path=/tmp/tomcat.xml'
```

> **service 指定运行状态**

- enabled：是否开机自动启动，取值为true或false
- name：服务名称
- state：状态，取位有started stopped restarted

> **shell：在远程主机上运行命令。**

> **script：将本地脚本复制到远程主机并运行**

> **yum 安装程序包**

- name：指明要安装的程序包，可以带上版本号
- state：present，latest安装 absent卸载

```
ansible test -m yum -a 'name=zsh state=present'
```


