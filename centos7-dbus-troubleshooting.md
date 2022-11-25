# Centos7 dbus问题总结


## Authorization not available. Check if polkit 

```
Authorization not available. Check if polkit service is running or see debug message for more information.

dbus.socket failed to listen on sockets: Address family not supported by protocol
Failed to listen on D-Bus System Message Bus Socket.
```
这个问题是因为dbus.socket状态异常，所有依赖dbus的启动都会去通过systemcall连接 dbus，当服务不可用时，所有服务无法以systemd方式正常启动/关闭。需要检查dbus.socket是否正常。本地使用需保证unix套接字的监听时启动的

## dbus重启导致登陆慢

```
systemd-logind: Failed to connect to system bus: Connection refused
systemd-logind: Failed to fully start up daemon: Connection refused
systemd: systemd-logind.service: main process exited, code=exited, status=1/FAILURE
systemd: Unit systemd-logind.service entered failed state.
systemd: systemd-logind.service failed.
systemd: systemd-logind.service has no holdoff time, scheduling restart.
systemd: start request repeated too quickly for systemd-logind.service
systemd: Unit systemd-logind.service entered failed state.
systemd: systemd-logind.service failed.

dbus[7782]: [system] Failed to activate service 'org.freedesktop.login1': timed out
dbus-daemon: dbus[7782]: [system] Failed to activate service 'org.freedesktop.login1':   timed out
```

参考：[ssh登陆缓慢](https://serverfault.com/questions/707377/slow-ssh-login-activation-of-org-freedesktop-login1-timed-out)

systemd-logind主要功能是为每一个登陆session创建一个systemd角度的cgroup管理对象，更方便对session使用cgroup，在dbus服务异常时，systemd-logind会导致登陆缓慢，并不影响正常登陆和ssh登陆。重启dbus.socket后需要也重启systemd-logind

## dbus开启远程连接

编辑 `/usr/share/dbus-1/system.conf` 或 `/etc/dbus-1/session.conf`

```
<listen>tcp:host=<ip>,bind=*,port=<port>,family=ipv4</listen>
<listen>unix:path=/run/user/<username>/dbus/user_bus_socket</listen>
<listen>unix:tmpdir=/tmp</listen>

<auth>ANONYMOUS</auth>
<allow_anonymous/>
```

参考：[dbus-send使用](https://blog.fpmurphy.com/2018/10/using-the-d-bus-interface-to-firewalld.html)

参考：[Linux DBus远程TCP连接失败](https://stackoverflow.com/questions/61327052/linux-dbus-remote-tcp-connection-with-systemd-fails)

## dbus faq

[faq](https://dbus.freedesktop.org/doc/)
