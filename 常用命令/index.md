# 

```sh
tail -f /data/logs/nginx/logs.go.uploadlog.com.log|awk &#39;$3 == &#34;203.90.248.130&#34; {print $0}&#39;
```



`resize2fs`  **调整ext文件系统的空间大小**

blkid 显示块设备属性



bash中的瑞士军刀 compgen&amp;complete&amp;compopt



compgen&amp;complete&amp;compopt是bash内置命令，虽说man给出的解释很少，但实际上功能很强大。

compgen于列出可以在Linux系统中执行的所有命令，但不仅限于这些功能



查看所有命令compgen -c

```
[root@node01 ~]# compgen -c|
cp
egrep
fgrep
grep
l.
ll
ls
mv
rm
which
...
bc
dc
go
gofmt
```

列出bash别名compgen -a

```
[root@node01 ~]# alias 
alias cp=&#39;cp -i&#39;
alias egrep=&#39;egrep --color=auto&#39;
alias fgrep=&#39;fgrep --color=auto&#39;
alias grep=&#39;grep --color=auto&#39;
alias l.=&#39;ls -d .* --color=auto&#39;
alias ll=&#39;ls -l --color=auto&#39;
alias ls=&#39;ls --color=auto&#39;
alias mv=&#39;mv -i&#39;
alias rm=&#39;rm -i&#39;
alias which=&#39;alias | /usr/bin/which --tty-only --read-alias --show-dot --show-tilde&#39;
[root@node01 ~]# compgen -a
cp
egrep
fgrep
grep
l.
ll
ls
mv
rm
which
```

查看内置bash命令

```
[root@node01 ~]# compgen -b
.
:
[
alias
bg
bind
break
builtin
caller
cd
command
compgen
complete
compopt
```

列出bash的保留字compgen -k

列出所有用户 compgen -u

—A action支持的action

| 操作      | 替代命令 | 说明                         |
| --------- | -------- | ---------------------------- |
| alias     | -a       | 别名                         |
| directory | -d       | 目录名称                     |
| file      | -f       | 文件名称                     |
| enabled   |          | 启用的shell内置项的名称      |
| disable   |          | 禁用的shell内置项的名称      |
| export    | -e       | 导出shell环境变量名称        |
| group     | -g       | 组名                         |
| user      | -u       | 用户名                       |
| hostname  |          | 取出HOSTFILE中所有hostname   |
| function  |          | 取出当前shell中所有函数名    |
| keyword   | -k       | 取出shell保留字              |
| service   | -s       | 取出service名称              |
| signal    |          | 取出信号名                   |
| variable  | -v       | 取出当前shell 环境所有变量名 |
| builtin   | -b       | 取出shell内建命令            |
| helptopic |          | 取出bash内建的帮助主题       |
| job       | -j       | 取出后台作业名称             |
| running   |          | 取出正在运行的作业的名称     |
|           |          |                              |
|           |          |                              |



| 选项         | 说明                                                         |      |
| ------------ | ------------------------------------------------------------ | ---- |
| -P prefix    | 当使用了其他选项时，为其他选项输出的结果添加前缀             |      |
| -S suffix    | 当使用了其他选项时，为其他选项输出的结果添加后缀             |      |
| -W wordlist  | 使用特殊符号对字符串进行拆分，匹配与对应的结果               |      |
| -X FilterPat | 用于和其他选项共同使用，前一个为数据源，将建一个选项的结果进行匹配，将匹配到的值删除，`!` 在表达式内是否定模式，动作为删除所有不配表达式的值。`!` 在命令行要加转义符 |      |
|              |                                                              |      |
|              |                                                              |      |
|              |                                                              |      |
|              |                                                              |      |
|              |                                                              |      |
|              |                                                              |      |
|              |                                                              |      |
|              |                                                              |      |
|              |                                                              |      |
|              |                                                              |      |
|              |                                                              |      |
|              |                                                              |      |
|              |                                                              |      |
|              |                                                              |      |
|              |                                                              |      |
|              |                                                              |      |
|              |                                                              |      |



