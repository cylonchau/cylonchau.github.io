# powercli创建虚拟机步骤及批量创建脚本


> 文档中心
> https://pubs.vmware.com/vsphere-51/index.jsp?topic=%2Fcom.vmware.powercli.cmdletref.doc%2FSet-OSCustomizationSpec.html
> https://blogs.vmware.com/PowerCLI/2014/05/working-customization-specifications-powercli-part-1.html
> http://powershelldistrict.com/powercli-oscustomizationspec/
> https://powercli-core.readthedocs.io/en/latest/cmd_new.html#new-oscustomizationspec
> [官方文档中心](https://pubs.vmware.com/vsphere-51/index.jsp?topic=%2Fcom.vmware.powercli.cmdletref.doc%2FSet-OSCustomizationSpec.html)

添加一块新硬盘

```
get-vm {vmname}|New-HardDisk -CapacityGB 300
```

批量设置硬件

```
Get-VM -name {hostname}*|set-VM -MemoryGB 4 -NumCPU 2
```

基于模板创建虚拟机

```
new-VM -Name {hostname}  -Template template-centos76 -VMHost 10.112.131.5  -OSCustomizationspec TestLinux
```

设置规范模板

```

Get-OSCustomizationSpec TestLinux|Get-OSCustomizationNicMapping|Set-OSCustomizationNicMapping \
-IpMode UseStaticIP \
-SubnetMask 255.255.255.0 \
-DefaultGateway 10.11.121.254 \
-IpAddress 10.11.121.203

Get-OSCustomizationSpec TestLinux|Get-OSCustomizationNicMapping|Set-OSCustomizationNicMapping \
-IpMode UseStaticIP \
-SubnetMask 255.255.255.0 \
-DefaultGateway 10.11.131.254 \
-IpAddress 10.11.131.64
```

获得规范信息

```
Get-OSCustomizationSpec liunx
```

创建一个新的规范

```
New-OSCustomizationSpec -OSType "Linux" -Name linux1 -Type Persistent –DnsServer "8.8.8.8"  –Domain "vmware.com"
```

批量创建脚本

```
$vserver="10.111.111.22"
$vuser="youname"
$vpwd="yourpasswd"


filter Convert-IP2Decimal 
{
    ([IPAddress][String]([IPAddress]$_)).Address
}

filter Convert-Decimal2IP
{
	([System.Net.IPAddress]$_).IPAddressToString
}

$IPPrefix="10.111.31"
$startIp=$args[0]
$startHostname=$args[1]
$num=$args[2]
$gateway="10.111.31.254"
$netmask="255.255.255.0"
$cpu=4
$mem=8
$template="template-centos7"

$vhost17=,"10.111.17.1","10.111.17.2","10.111.17.3","10.111.17.4"
$vhost20=,"10.111.20.1","10.111.20.2","10.111.20.3","10.111.20.4"
$vhost31=,"10.111.31.1","10.111.31.2","10.111.31.3","10.111.31.4"


function LoginVSphere($server,$user,$pwd){
	Connect-VIServer -Server $server[0] -User $server[1] -Password $server[2] -EA "Stop"
}

LoginVSphere($vserver, $vuser, $vpwd)
$temp=Get-OSCustomizationSpec TestLinux 






for($n=[int]$startIp;$n -le [int]$num; $n++)
{
	$ipaddr=-Join($IPPrefix,'.',$n)
	$hostname=-Join($startHostname,$n)
	$temp|Get-OSCustomizationNicMapping|Set-OSCustomizationNicMapping -IpMode UseStaticIP -SubnetMask $netmask -DefaultGateway $gateway -IpAddress $ipaddr
	new-VM -Name $hostname -Template $template -VMHost 10.111.31.5 -OSCustomizationspec TestLinux -Confirm:$false
	Get-VM -Name $hostname|Set-VM -MemoryGB $mem -NumCPU $cpu -Confirm:$false
}
```

 

> **Reference**
> https://communities.vmware.com/thread/567585
> [脚本参考](https://www.itnotebooks.com/?p=716)
> https://blogs.vmware.com/PowerCLI/2014/05/working-customization-specifications-powercli-part-1.html
> http://powershelldistrict.com/powercli-oscustomizationspec/
> https://pubs.vmware.com/vsphere-51/index.jsp?topic=%2Fcom.vmware.powercli.cmdletref.doc%2FSet-OSCustomizationSpec.html

 
