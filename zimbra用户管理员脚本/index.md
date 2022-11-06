# zimbra用户管理员脚本




```sh
#!/bin/bash
# $1 domain
# $2 email
zmprov ma $2 zimbraIsDelegatedAdminAccount TRUE
zmprov ma $2 zimbraAdminConsoleUIComponents cartBlancheUI zimbraAdminConsoleUIComponents domainListView zimbraAdminConsoleUIComponents accountListView zimbraAdminConsoleUIComponents DLListView
zmprov ma $2 zimbraDomainAdminMaxMailQuota 0
zmprov grantRight domain $1 usr $2 &#43;createAccount
zmprov grantRight domain $1 usr $2 &#43;createAlias
zmprov grantRight domain $1 usr $2 &#43;createCalendarResource
zmprov grantRight domain $1 usr $2 &#43;createDistributionList
zmprov grantRight domain $1 usr $2 &#43;deleteAlias
zmprov grantRight domain $1 usr $2 &#43;listDomain
zmprov grantRight domain $1 usr $2 &#43;domainAdminRights
zmprov grantRight domain $1 usr $2 set.account.zimbraAccountStatus
zmprov grantRight domain $1 usr $2 set.account.sn
zmprov grantRight domain $1 usr $2 set.account.displayName
zmprov grantRight domain $1 usr $2 set.account.zimbraPasswordMustChange
zmprov grantRight account $2 usr $2 &#43;deleteAccount
zmprov grantRight account $2 usr $2 &#43;getAccountInfo
zmprov grantRight account $2 usr $2 &#43;getAccountMembership
zmprov grantRight account $2 usr $2 &#43;listAccount
zmprov grantRight account $2 usr $2 &#43;removeAccountAlias
zmprov grantRight account $2 usr $2 &#43;renameAccount
zmprov grantRight account $2 usr $2 &#43;setAccountPassword
zmprov grantRight account $2 usr $2 &#43;viewAccountAdminUI
zmprov grantRight account $2 usr $2 &#43;configureQuota
```
