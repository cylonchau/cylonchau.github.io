# 


```pwsh
Connect-VIServer -Server 10.11.17.20 -User administrator@vsphere.local -Password YaboDC02@123456

Get-OSCustomizationSpec linux1|Get-OSCustomizationNicMapping|Set-OSCustomizationNicMapping -IpMode UseStaticIP -SubnetMask 255.255.255.0 -DefaultGateway 10.10.12.254 -IpAddress 10.10.12.114

new-VM -Name zy-ntw-yabo-prod-dbvm-pushredis01  -Template template-centos76 -VMHost 10.10.12.14  -OSCustomizationspec linux1

Get-VM -name zy-ntw-yabo-prod-dbvm-pushredis01|set-VM -MemoryGB 96 -NumCPU 16

	
get-vm zy-ntw-yabo-prod-dbvm-pushredis01|New-HardDisk -CapacityGB 100

New-Snapshot -Name "20200622"


Connect-VIServer -Server 10.11.17.20 -User administrator@vsphere.local -Password YaboDC@123456


Connect-VIServer -Server 10.11.17.20 -User administrator@vsphere.local -Password YaboDC02@123456


Connect-VIServer -Server 10.10.200.20 -User administrator@vsphere.local -Password YaboDC@123456


Get-OSCustomizationSpec linux1|Get-OSCustomizationNicMapping|Set-OSCustomizationNicMapping -IpMode UseStaticIP -SubnetMask 255.255.255.0 -DefaultGateway 10.11.31.254 -IpAddress 10.11.31.65

new-VM -Name zy-ntw-zr-prod-vip-upgrade01 -Template template-centos76 -VMHost 10.11.31.4 -OSCustomizationspec linux1

Get-OSCustomizationSpec linux1|Get-OSCustomizationNicMapping|Set-OSCustomizationNicMapping -IpMode UseStaticIP -SubnetMask 255.255.255.0 -DefaultGateway 10.11.31.254 -IpAddress 10.11.31.66

new-VM -Name zy-ntw-zr-prod-vip-upgrade02 -Template template-centos76 -VMHost 10.11.31.5 -OSCustomizationspec linux1


get-vm zy-ntw-zr-prod-vip-upgrade*|set-VM -MemoryGB 8 -NumCPU 4

get-harddisk zy-ntw-zr-prod-vip-upgrade*      
get-harddisk zy-ntw-zr-prod-vip-upgrade*|set-harddisk -CapacityGB 100 
# 现有硬盘扩容
get-vm zy-ntw-zr-prod-vip-upgrade* |get-harddisk|set-harddisk -CapacityGB 100

## 删除虚拟机
remove-vm –deletepermanently 
```


