# 如何通过源码编译Kubernetes


## 本地构建
### 选择要构建的版本

```
git checkout tags/v1.19.5
```

### 将依赖包复制到对应路径下

```
cp staging/src/k8s.io vendor/
```

### 调整makefile

在windows上编译的克隆下可能文件编码变了，需要手动修改下文件编码。比如说出现 `\r not found` 类似关键词时

这里转换编码使用了 dos2unix，需要提前安装下

```
apt install dos2unix
```
转换原因是因为对于bash 脚本执行识别不了windows的换行
```
find . -name '*.sh' -exec dos2unix {} \;
```

然后将 `build/root/` 的文件复制到项目根目录

```
cp build/root/Makefile* ./
```

### 编译
查看帮助 `make help`

编译 `make all WHAT=cmd/kube-apiserver GOFLAGS=-v`

`WHAT=cmd/kube-apiserver` 为仅编译单一组件，`all` 为所有的组件

还可以增加其他的一些环境变量 `KUBE_BUILD_PLATFORMS=` 如编译的平台

更多的可以 `make help` 查看帮助

### 编译中问题

**Makefile:93: recipe for target 'all' failed**

```
!!! [0515 21:32:52] Call tree:
!!! [0515 21:32:52]  1: /mnt/d/src/go_work/src/kubernetes/hack/lib/golang.sh:717 kube::golang::build_some_binaries(...)
!!! [0515 21:32:52]  2: /mnt/d/src/go_work/src/kubernetes/hack/lib/golang.sh:861 kube::golang::build_binaries_for_platform(...)
!!! [0515 21:32:52]  3: hack/make-rules/build.sh:27 kube::golang::build_binaries(...)
!!! [0515 21:32:52] Call tree:
!!! [0515 21:32:52]  1: hack/make-rules/build.sh:27 kube::golang::build_binaries(...)
!!! [0515 21:32:52] Call tree:
!!! [0515 21:32:52]  1: hack/make-rules/build.sh:27 kube::golang::build_binaries(...)
Makefile:93: recipe for target 'all' failed
```
这里看报错根本不知道发生什么问题，使用 `strace` 追送了下，很明显看到是没有gcc

cgo: exec gcc: exec: "gcc": executable file not found in $PATH

```
rt_sigprocmask(SIG_BLOCK, [HUP INT QUIT TERM XCPU XFSZ], NULL, 8) = 0
clone(child_stack=NULL, flags=CLONE_CHILD_CLEARTID|CLONE_CHILD_SETTID|SIGCHLD, child_tidptr=0x7fbf45410a10) = 17890
rt_sigprocmask(SIG_SETMASK, [], NULL, 8) = 0
wait4(-1, +++ [0515 21:34:40] Building go targets for linux/amd64:
    cmd/kubelet
k8s.io/kubernetes/vendor/github.com/opencontainers/runc/libcontainer/system
k8s.io/kubernetes/vendor/github.com/mindprince/gonvml
# k8s.io/kubernetes/vendor/github.com/opencontainers/runc/libcontainer/system
cgo: exec gcc: exec: "gcc": executable file not found in $PATH
# k8s.io/kubernetes/vendor/github.com/mindprince/gonvml
cgo: exec gcc: exec: "gcc": executable file not found in $PATH
!!! [0515 21:34:42] Call tree:
!!! [0515 21:34:42]  1: /mnt/d/src/go_work/src/kubernetes/hack/lib/golang.sh:717 kube::golang::build_some_binaries(...)
!!! [0515 21:34:42]  2: /mnt/d/src/go_work/src/kubernetes/hack/lib/golang.sh:861 kube::golang::build_binaries_for_platform(...)
!!! [0515 21:34:42]  3: hack/make-rules/build.sh:27 kube::golang::build_binaries(...)
!!! [0515 21:34:42] Call tree:
!!! [0515 21:34:42]  1: hack/make-rules/build.sh:27 kube::golang::build_binaries(...)
!!! [0515 21:34:42] Call tree:
!!! [0515 21:34:42]  1: hack/make-rules/build.sh:27 kube::golang::build_binaries(...)
[{WIFEXITED(s) && WEXITSTATUS(s) == 1}], 0, NULL) = 17890
--- SIGCHLD {si_signo=SIGCHLD, si_code=CLD_EXITED, si_pid=17890, si_uid=0, si_status=1, si_utime=0, si_stime=0} ---
rt_sigreturn({mask=[]})                 = 17890
openat(AT_FDCWD, "/usr/share/locale/C.UTF-8/LC_MESSAGES/make.mo", O_RDONLY) = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/usr/share/locale/C.utf8/LC_MESSAGES/make.mo", O_RDONLY) = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/usr/share/locale/C/LC_MESSAGES/make.mo", O_RDONLY) = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/usr/share/locale-langpack/C.UTF-8/LC_MESSAGES/make.mo", O_RDONLY) = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/usr/share/locale-langpack/C.utf8/LC_MESSAGES/make.mo", O_RDONLY) = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/usr/share/locale-langpack/C/LC_MESSAGES/make.mo", O_RDONLY) = -1 ENOENT (No such file or directory)
fstat(1, {st_mode=S_IFCHR|0640, st_rdev=makedev(4, 1), ...}) = 0
ioctl(1, TCGETS, {B38400 opost isig icanon echo ...}) = 0
write(1, "Makefile:93: recipe for target '"..., 44Makefile:93: recipe for target 'all' failed
) = 44
write(2, "make: *** [all] Error 1\n", 24make: *** [all] Error 1
) = 24
rt_sigprocmask(SIG_BLOCK, [HUP INT QUIT TERM XCPU XFSZ], NULL, 8) = 0
rt_sigprocmask(SIG_SETMASK, [], NULL, 8) = 0
chdir("/mnt/d/src/go_work/src/kubernetes") = 0
close(1)                                = 0
exit_group(2)                           = ?
+++ exited with 2 +++
```
**修改后编译问题可以明显看出是哪里**

如尝试增加一种资源类型后编译，这种类型的错误可以根据报错提示进行修改

![](https://cdn.jsdelivr.net/gh/CylonChau/imgbed//img/1380340-20220516172727798-791337963.png)

```
+++ [0515 21:47:59] Building go targets for linux/amd64:
    cmd/kube-apiserver
k8s.io/kubernetes/vendor/k8s.io/api/apps/v1
# k8s.io/kubernetes/vendor/k8s.io/api/apps/v1
vendor/k8s.io/api/apps/v1/register.go:48:3: cannot use &StateDeploy{} (type *StateDeploy) as type runtime.Object in argument to scheme.Ad
dKnownTypes:
        *StateDeploy does not implement runtime.Object (missing DeepCopyObject method)
!!! [0515 21:48:01] Call tree:
!!! [0515 21:48:01]  1: /mnt/d/src/go_work/src/kubernetes/hack/lib/golang.sh:706 k
```

