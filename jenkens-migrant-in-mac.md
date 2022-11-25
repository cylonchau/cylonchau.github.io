# jenkins在Mac OS下的迁移记录


修改启动用户

先停止jenkins服务


```sh
sudo launchctl unload /Library/LaunchDaemons/org.jenkins-ci.plist
sudo vim /Library/LaunchDaemons/org.jenkins-ci.plist
```
授权jenkins工作目录和临时目录

```
sudo chown -R zhulangren:wheel /Users/Shared/Jenkins/
sudo chown -R zhulangren:wheel /var/log/jenkins/
```
启动jenkins

```
sudo launchctl load /Library/LaunchDaemons/org.jenkins-ci.plist
```

jenkins自启动文件路径


```
/Library/LaunchDaemons/org.jenkins-ci.plist
```
卸载脚本文件

```
/Library/Application\ Support/Jenkins/Uninstall.command
```
修改jenkins启动端口

```
sudo defaults write /Library/Preferences/org.jenkins-ci httpPort '9999'
```

读取jenkins配置文件

```
defaults read /Library/Preferences/org.jenkins-ci
```

设置自启动


```
sudo launchctl load /Library/LaunchDaemons/org.jenkins-ci.plis
```

取消自启动

```
sudo launchctl unload /Library/LaunchDaemons/org.jenkins-ci.plist
```
jenkins war路径


```
/Applications/Jenkins/jenkins.war
```



> Reference
>
> [Mac Jenkins 权限问题](https://www.cnblogs.com/ihojin/p/jenkins-permission.html) 
