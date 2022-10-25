# 使用eNSP构建calico BGP网络




![image-20210129230307007](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20210129230307007.png)

实验文件： [calico BGP.zip](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/calico BGP.zip) 

R1

```
system-view 
sysname R1

interface l0
ip address 1.1.1.1 32

interface g0/0/0
ip address 10.1.0.1 24

interface g0/0/1
ip address 10.3.0.1 24

bgp 100
router-id 1.1.1.1
peer 10.1.0.2 as-number 123
peer 10.3.0.2 as-number 456
dis this
dis ip interface b
```

R2

```
system-view 
sysname R2

interface l0
ip address 2.2.2.2 32

interface g0/0/0
ip address 10.2.0.1 24

interface g0/0/1
ip address 10.4.0.1 24

bgp 200
router-id 2.2.2.2
peer 10.4.0.2 as-number 123
peer 10.2.0.2 as-number 456
dis ip interface b
```

从下至上配置

R3

```
system-view 
sysname R3

interface l0
ip address 3.3.3.3 32

interface g0/0/0
ip address 10.1.0.2 24

interface g0/0/1
ip address 10.5.0.1 24

vlan 2
int vlan 2
ip address 10.6.0.1 24
in e0/0/0
port link-type access 
port default vlan 2
dis ip interface brief

vlan 3
int vlan 3
ip address 10.4.0.2 24
in e0/0/1
port link-type access 
port default vlan 3
dis ip interface brief 

ospf router-id 3.3.3.3
area 0
network 10.1.0.0 0.0.0.255
network 10.5.0.0 0.0.0.255
network 3.3.3.3 0.0.0.0
network 10.4.0.0 0.0.0.255
network 10.6.0.0 0.0.0.255
dis this

bgp 123
router-id 3.3.3.3
peer 5.5.5.5 as-number 123
peer 5.5.5.5 connect-interface l0
peer 6.6.6.6 as-number 123
peer 6.6.6.6 connect-interface l0
dis this
peer 5.5.5.5 reflect-client
peer 6.6.6.6 reflect-client

peer 10.1.0.1 as-number 100
peer 10.4.0.1 as-number 200
dis this
```

R5

注意这里OSPF宣告的路由，OSPF优先级高于BGP，此处不能宣告`0.0.0.0 255.255.255.255`

```
system-view 
sysname R5

interface l0
ip address 5.5.5.5 32
quit

vlan 2
int vlan 2
ip address 10.5.0.2 24
in e0/0/0
port link-type access 
port default vlan 2

dis ip interface brief 

ospf router-id 5.5.5.5
area 0
network 5.5.5.5 0.0.0.0
network 10.5.0.0 0.0.0.255
dis this

bgp 123
router-id 5.5.5.5
peer 3.3.3.3 as-number 123
peer 3.3.3.3 connect-interface l0
dis this
```

R6

```
system-view 
sysname R6

interface l0
ip address 6.6.6.6 32
quit

vlan 2
int vlan 2
ip address 10.6.0.2 24
in e0/0/0
port link-type access 
port default vlan 2

dis ip interface brief 

ospf router-id 6.6.6.6
area 0
network 6.6.6.6 0.0.0.0
network 10.6.0.0 0.0.0.255
dis this

bgp 123
router-id 6.6.6.6
peer 3.3.3.3 as-number 123
peer 3.3.3.3 connect-interface l0
dis this
```

R4

```
system-view 
sysname R4

interface l0
ip address 4.4.4.4 32

interface g0/0/0
ip address 10.2.0.2 24

interface g0/0/1
ip address 10.7.0.1 24

vlan 2
int vlan 2
ip address 10.8.0.1 24
in e0/0/0
port link-type access 
port default vlan 2
dis ip interface brief

vlan 3
int vlan 3
ip address 10.3.0.2 24
in e0/0/1
port link-type access 
port default vlan 3
dis ip interface brief 

ospf router-id 4.4.4.4
area 0
network 10.2.0.0 0.0.0.255
network 10.3.0.0 0.0.0.255
network 4.4.4.4 0.0.0.0
network 10.7.0.0 0.0.0.255
network 10.8.0.0 0.0.0.255
dis this

bgp 456
router-id 4.4.4.4
peer 7.7.7.7 as-number 456
peer 7.7.7.7 connect-interface l0
peer 8.8.8.8 as-number 456
peer 8.8.8.8 connect-interface l0
dis this
peer 7.7.7.7 reflect-client
peer 8.8.8.8 reflect-client

peer 10.3.0.1 as-number 100
peer 10.2.0.1 as-number 200
dis this
```

R7

```
system-view 
sysname R7

interface l0
ip address 7.7.7.7 32
quit

vlan 2
int vlan 2
ip address 10.7.0.2 24
in e0/0/0
port link-type access 
port default vlan 2

dis ip interface brief 

ospf router-id 7.7.7.7
area 0
network 7.7.7.7 0.0.0.0
network 10.7.0.0 0.0.0.255
dis this

bgp 456
router-id 7.7.7.7
peer 4.4.4.4 as-number 456
peer 4.4.4.4 connect-interface l0
dis this
```

R8

```
system-view 
sysname R8

interface l0
ip address 8.8.8.8 32

interface g0/0/0
ip address 10.8.0.2 24

dis ip interface brief 

ospf router-id 8.8.8.8
area 0
network 8.8.8.8 0.0.0.0
network 10.8.0.0 0.0.0.255
dis this

bgp 456
router-id 8.8.8.8
peer 4.4.4.4 as-number 456
peer 4.4.4.4 connect-interface l0
dis this
```

在R7与R5各添加一条路由

```
interface L11
ip address 77.77.77.77 32
quit
bgp 456
network 77.77.77.77 255.255.255.255

interface L11
ip address 55.55.55.55 32
quit
bgp 123
network 55.55.55.55 255.255.255.255
```

可以看到R1 R2 与 R6 R8都通过对应的bgp协议学习到相应的路由。

```
[R2]dis bgp routing-table 

 BGP Local router ID is 2.2.2.2 
 Status codes: * - valid, > - best, d - damped,
               h - history,  i - internal, s - suppressed, S - Stale
               Origin : i - IGP, e - EGP, ? - incomplete

 Total Number of Routes: 4
      Network            NextHop        MED        LocPrf  PrefVal Path/Ogn
 *>   55.55.55.55/32     10.4.0.2                            0      123i
 *                       10.2.0.2                            0      456 100 123i
 *>   77.77.77.77/32     10.2.0.2                            0      456i
 *                       10.4.0.2                            0      123 100 456i


[R6-bgp]dis bgp routing-table 

 BGP Local router ID is 6.6.6.6 
 Status codes: * - valid, > - best, d - damped,
               h - history,  i - internal, s - suppressed, S - Stale
               Origin : i - IGP, e - EGP, ? - incomplete

 Total Number of Routes: 2
      Network            NextHop        MED        LocPrf    PrefVal Path/Ogn
 *>i  55.55.55.55/32     5.5.5.5         0          100        0      i
 *>i  77.77.77.77/32     10.1.0.1                   100        0      100 456i
[R6-bgp]
```

遇到问题：

vlan未启动，需要查看对应绑定的端口是否正确

ospf配置错误，`undo ospf 1 ` 重启ospf
