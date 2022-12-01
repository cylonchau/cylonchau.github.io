# centos安装powershell和powercli 


poershell [github](https://github.com/PowerShell/PowerShell/releases)

本次采用github下载对应的rpm进行安装

windows下安装方法

[离线安装包下载地址](https://code.vmware.com/web/tool/12.0.0/vmware-powercli)

```
yum install -y https://github.com/PowerShell/PowerShell/releases/download/v7.0.0/powershell-lts-7.0.0-1.rhel.7.x86_64.rpm
```

安装完成后再终端输入以下命令 `pwsh`。在一个PowerShell会话，您可以通过运行以下命令来检查的PowerShell的版本。

```
PS /root> $PSVersionTable.PSVersion

Major  Minor  Patch  PreReleaseLabel BuildLabel
-----  -----  -----  --------------- ----------
7      0      0  

Set-PSRepository -Name "PSGallery" -InstallationPolicy "Trusted"
```

接下来，运行以下命令来安装VMware.PowerCLI模块。这将发现和在PSGallery安装最新版本的模块

```
Find-Module "VMware.PowerCLI" | Install-Module -Scope "CurrentUser" -AllowClobber
```

完成后，通过运行以下命令进行检查，以确保该模块安装。
```

Get-Module "VMware.PowerCLI" -ListAvailable | FT -Autosize

 

PS /root> Get-Module "VMware.PowerCLI" -ListAvailable | FT -Autosize


    Directory: /root/.local/share/powershell/Modules

ModuleType Version         PreRelease Name            PSEdition ExportedCommands
---------- -------         ---------- ----            --------- ----------------
Manifest   12.0.0.15947286            VMware.PowerCLI Desk 
```

 如果你想看到所有的VMware安装的模块，运行以下命令。

```
PS /root> Get-Module "VMware.*" -ListAvailable | FT -Autosize


    Directory: /root/.local/share/powershell/Modules

ModuleType Version         PreRelease Name                                    PSEdition ExportedCommands
---------- -------         ---------- ----                                    --------- ----------------
Script     12.0.0.15947289            VMware.CloudServices                    Desk      {Connect-Vcs, Get-VcsOrganizationRole, Get-VcsService, Get-VcsServiceRole…
Script     7.0.0.15902843             VMware.DeployAutomation                 Desk      {Add-DeployRule, Add-ProxyServer, Add-ScriptBundle, Copy-DeployRule…}
Script     7.0.0.15902843             VMware.ImageBuilder                     Desk      {Add-EsxSoftwareDepot, Add-EsxSoftwarePackage, Compare-EsxImageProfile, E…
Manifest   12.0.0.15947286            VMware.PowerCLI                         Desk      
Script     7.0.0.15939650             VMware.Vim                              Desk      
Script     12.0.0.15939657            VMware.VimAutomation.Cis.Core           Desk      {Connect-CisServer, Disconnect-CisServer, Get-CisService}
Script     12.0.0.15940183            VMware.VimAutomation.Cloud              Desk      {Add-CIDatastore, Connect-CIServer, Disconnect-CIServer, Get-Catalog…}
Script     12.0.0.15939652            VMware.VimAutomation.Common             Desk      {Get-Task, New-OAuthSecurityContext, Stop-Task, Wait-Task}
Script     12.0.0.15939655            VMware.VimAutomation.Core               Desk      {Add-PassthroughDevice, Add-VirtualSwitchPhysicalNetworkAdapter, Add-VMHo…
Script     12.0.0.15939647            VMware.VimAutomation.Hcx                Desk      {Connect-HCXServer, Disconnect-HCXServer, Get-HCXAppliance, Get-HCXComput…
Script     7.12.0.15718406            VMware.VimAutomation.HorizonView        Desk      {Connect-HVServer, Disconnect-HVServer}
Script     12.0.0.15939670            VMware.VimAutomation.License            Desk      Get-LicenseDataManager
Script     12.0.0.15939671            VMware.VimAutomation.Nsxt               Desk      {Connect-NsxtServer, Disconnect-NsxtServer, Get-NsxtPolicyService, Get-Ns…
Script     12.0.0.15939651            VMware.VimAutomation.Sdk                Desk      {Get-ErrorReport, Get-PSVersion, Get-InstallPath}
Script     12.0.0.15939672            VMware.VimAutomation.Security           Desk      {Add-AttestationServiceInfo, Add-KeyProviderServiceInfo, Add-TrustAuthori…
Script     11.5.0.14899557            VMware.VimAutomation.Srm                Desk      {Connect-SrmServer, Disconnect-SrmServer}
Script     12.0.0.15939648            VMware.VimAutomation.Storage            Desk      {Add-EntityDefaultKeyProvider, Add-KeyManagementServer, Add-VsanFileServi…
Script     1.3.0.0                    VMware.VimAutomation.StorageUtility     Desk      Update-VmfsDatastore
Script     12.0.0.15940185            VMware.VimAutomation.Vds                Desk      {Add-VDSwitchPhysicalNetworkAdapter, Add-VDSwitchVMHost, Export-VDPortGro…
Script     12.0.0.15947287            VMware.VimAutomation.Vmc                Desk      {Add-VmcSddcHost, Connect-Vmc, Disconnect-Vmc, Get-AwsAccount…}
Script     12.0.0.15940184            VMware.VimAutomation.vROps              Desk      {Connect-OMServer, Disconnect-OMServer, Get-OMAlert, Get-OMAlertDefinitio…
Script     12.0.0.15947288            VMware.VimAutomation.WorkloadManagement Desk      {Get-WMNamespace, Get-WMNamespacePermission, Get-WMNamespaceStoragePolicy…
Script     6.5.1.7862888              VMware.VumAutomation                    Desk      {Add-EntityBaseline, Copy-Patch, Get-Baseline, Get-Compliance…}
```

当VMware.PowerCLI发布新版本，你可以运行下面的命令来更新。

```
Update-Module "VMware.PowerCLI"
```

VMware.PowerCLI现在已安装完成，可以连接到您的vCenter Server或ESXi主机

```
PS /root> Set-PowerCLIConfiguration -InvalidCertificateAction "Ignore"

Perform operation?
Performing operation 'Update PowerCLI configuration.'?
[Y] Yes  [A] Yes to All  [N] No  [L] No to All  [S] Suspend  [?] Help (default is "Y"): Y

Scope    ProxyPolicy     DefaultVIServerMode InvalidCertificateAction  DisplayDeprecationWarnings WebOperationTimeout
                                                                                                  Seconds
-----    -----------     ------------------- ------------------------  -------------------------- -------------------
Session  UseSystemProxy  Multiple            Ignore                    True                       300
User                                         Ignore                                               
AllUsers  
```
