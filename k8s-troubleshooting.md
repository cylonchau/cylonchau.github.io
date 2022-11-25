# Kubernetes安装错误


apiserver报错 没有kubeappserver用户

```
Failed at step USER spawning  No such process
```



```log
Mar  2 04:54:56 node02 systemd: controller-manager.service: main process exited, code=exited, status=1/FAILURE
Mar  2 04:54:56 node02 kube-controller-manager: Get http://127.0.0.1:8443/api/v1/namespaces/kube-system/configmaps/extension-apiserver-authentication: net/http: HTTP/1.x transport connection broken: malformed HTTP response "\x15\x03\x01\x00\x02\x02"
Mar  2 04:54:56 node02 systemd: Unit controller-manager.service entered failed state.
```


[Unable to connect to the server: net/http: HTTP/1.x transport connection broken: malformed HTTP resp - kevin_loving的博客 - CSDN博客](https://blog.csdn.net/kevin_loving/article/details/81326470?utm_source=blogxgwz9)


可能是启动单元文件有问题，手动启动后正常

```
Mar  2 09:44:51 node02 kube-controller-manager: W0302 09:44:51.672404   39926 client_config.go:554] error creating inClusterConfig, falling 
back to default config: unable to load in-cluster configuration, KUBERNETES_SERVICE_HOST and KUBERNETES_SERVICE_PORT must be defined
```

[创建 Kubernetes 集群：配置 Master 组件 \| Kuops Blog](https://kuops.com/2018/07/19/deploy-kubernets-ha-06/#Kube-apiserver-部分启动参数说明)

```log
E0303 16:43:32.438966   44108 leaderelection.go:270] error retrieving resource lock kube-system/kube-controller-manager: Get https://127.0.0.1:8443/api/v1/namespaces/kube-system/endpoints/kube-controller-manager?timeout=10s: x509: certificate signed by unknown authority。
```






```
kubectl config set-cluster k8s \
--certificate-authority=/etc/k8s/ca.crt \
--embed-certs=true  \
--server=https://127.0.0.1:8443 \
--kubeconfig=/etc/k8s/controller-manager
```


```
kubectl config set-credentials system:controller-manager \
--client-certificate=/etc/k8s/sa.crt \
--client-key=/etc/k8s/sa.key  \
--embed-certs=true \
--kubeconfig=/etc/k8s/controller-manager
```





kubectl config set-credentials testrbac --client-certificate=/etc/kubernetes/pki/testrbac.crt --client-key=/etc/kubernetes/pki/testrbac.key --embed-certs=true  --kubeconfig=testrbac



```bash
kubectl config set-context system:controller-manager@k8s \
--cluster=k8s \
--user=system:controller-manager \
--kubeconfig=/etc/k8s/controller-manager

$
$
$
$kubectl config use-context  system:controller-manager@k8s  --kubeconfig=/etc/k8s/controller-manager
Switched to context "system:controller-manager@k8s".
```



生成admin


kubectl config set-cluster k8s --certificate-authority=/etc/k8s/ca.crt --embed-certs=true  --server=admin --kubeconfig=/etc/k8s/admin

kubectl config set-credentials admin --client-certificate=/etc/k8s/admin.crt --client-key=/etc/k8s/admin.key --embed-certs=true --kubeconfig=/etc/k8s/admin


kubectl config set-context admin@k8s --cluster=k8s --user=admin --kubeconfig=/etc/k8s/admin

kubectl config use-context admin@k8s --kubeconfig=/etc/k8s/admin





kubectl config set-cluster k8s --certificate-authority=ca.crt  --embed-certs=true --server=https://10.0.0.6:8443 --kubeconfig=/etc/k8s/bootstrap-kubelet.conf

kubectl config set-credentials tls-bootstrap-token-user --token=${BOOTSTRAP_TOKEN} --kubeconfig=/etc/k8s/bootstrap-kubelet.conf

kubectl config set-context tls-bootstrap-token-user@k8s --cluster=k8s --user=tls-bootstrap-token-user --kubeconfig=/etc/k8s/bootstrap-kubelet.conf

kubectl config use-context tls-bootstrap-token-user@k8s --kubeconfig=/etc/k8s/bootstrap-kubelet.conf





```
Nov 14 16:31:03 master01 kubelet: E1114 16:31:03.677280    8587 kubelet.go:2270] node "master01" not found
```



```
k8s.io/client-go/informers/factory.go:135: Failed to list *v1.CSIDriver: csidrivers.storage.k8s.io is forbidden: User "system:anonymous" cannot list resource "csidrivers" in API group "storage.k8s.io" at the cluster scope
```



```
380   19810 kubelet.go:2292] node "master-machine" not found
May 12 23:45:11 master-machine kubelet: E0512 23:45:11.415099   19810 kubelet.go:2292] node "master-machine" not found
May 12 23:45:11 master-machine kubelet: E0512 23:45:11.460097   19810 certificate_manager.go:434] Failed while requesting a signed certificate from the master: cannot create certificate signing request: certificatesigningrequests.certificates.k8s.io is forbidden: User "system:anonymous" cannot create resource "certificatesigningrequests" in API group "certificates.k8s.io" at the cluster scope
```



> 问题：`"master01" is forbidden: User "system:anonymous" `
>
> 原因：用户未授权
>
> 解决  ：`kubectl create clusterrolebinding kube-apiserver:kubelet-apis --clusterrole=system:kubelet-api-admin --user kubernetes-master` && `kubectl certificate approve`

```
master01 kubelet: E1114 17:16:33.581116    8559 kubelet.go:2270] node "master01" not found

failed to ensure node lease exists, will retry in 6.4s, error: leases.coordination.k8s.io "master01" is forbidden: User "system:anonymous" cannot get resource "leases" in API group "coordination.k8s.io" in the namespace "kube-node-lease"
```

>问题1： flannela访问`dial tcp 10.96.0.1:443: connect: connection timed out` 
>
>问题2：`open /run/flannel/subnet.env: no such file or directory`
>
>问题3： `Container runtime network not ready: NetworkReady=false reason:NetworkPluginNotReady message:docker: network plugin is not ready: cni config uninitialized`
>
>解决：检查kube-proxy组件

```
Nov 14 17:30:04 master01 kubelet: E1114 17:30:04.097261    1160 kubelet.go:2190] Container runtime network not ready: NetworkReady=false reason:NetworkPluginNotReady message:docker: network plugin is not ready: cni config uninitialized
```

```
[root@master01 kubernetes]# kubectl logs kube-flannel-ds-dvsv7 -n kube-system
....
E1114 23:22:20.984238       1 main.go:243] Failed to create SubnetManager: error retrieving pod spec for 'kube-system/kube-flannel-ds-dvsv7': Get "https://10.96.0.1:443/api/v1/namespaces/kube-system/pods/kube-flannel-ds-dvsv7": dial tcp 10.96.0.1:443: connect: connection timed out
[root@master01 kubernetes]# 
```

```
create pod sandbox: rpc error: code = unknown desc = failed to set up sandbox container : networkplugin cni failed to set up pod "coredns-8686db44f5-tgpp4_kube-system" network: open /run/flannel/subnet.env: no such file or directory
```

> 问题：kubectl get csr 显示No Resources Found的解决记录
>
> 解决：<font size=2>需要先将 bootstrap token 文件中的 kubelet-bootstrap 用户赋予 system:node-bootstrapper 角色，然后 kubelet 才有权限创建认证请求</font>



```
{"log":"E1204 15:10:26.155093       1 main.go:234] Failed to create SubnetManager: error retrieving pod spec for 'kube-system/kube-flannel-ds-9pmsp': Get \"https://192.160.0.1:443/api/v1/namespaces/kube-system/pods/kube-flannel-ds-9pmsp\": x509: certificate is valid for 192.168.0.1, 10.0.0.5, not 192.160.0.1\n","stream":"stderr","time":"2021-12-04T15:10:26.156115275Z"}
```





https://blog.51cto.com/juestnow/2439614

```log
runtime network not ready: NetworkReady=false reason:NetworkPluginNotReady message:docker: network plugin is not ready: cni config uninitialized
```



## 授予KUBE-APISERVER对KUBELET API的访问权限

kubelet启动启动时，`--kubeletconfig` 使用参数对应的文件是否存在 如果不存在`--bootstrap-kubeconfig`指定的kubeconfig文件向kube-apiserver发送CSR请求

kube-apiserver 收到 CSR 请求后，对其中的 token 进行认证，认证通过后将请求的用户设置为`system:bootstrap:<Token ID>`，组设置为 ，`system:bootstrappers`此操作称为`Bootstrap Token Auth`。

默认这个用户和组没有创建CSR的权限，kubelet没有启动，错误日志如下：

```
Unable to register node "master-machine" with API server: nodes is forbidden: User "system:anonymous" cannot create resource "nodes" in API group "" at the cluster scope
```

解决方法是：创建一个集群角色绑定，绑定一个组：`system:bootstrapper` 和一个clusterrole `system:node-bootstrapper`

```
$ kubectl create clusterrolebinding kubelet-bootstrap --clusterrole=system:node-bootstrapper --group=system:bootstrappers
```



kubectl create clusterrolebinding kubelet-bootstrap --clusterrole=system:node-bootstrapper --group=system:bootstrappers

system:serviceaccount:default:default



kubectl create clusterrolebinding firewalld-default --clusterrole=system:aggregate-to-admin --user=system:serviceaccount:default:default

system:aggregate-to-admin





Jun 22 16:53:10 master-machine kube-apiserver: E0622 16:53:10.620143   28763 controller.go:114] loading OpenAPI spec for "v1.firewalld.fedoraproject.org" failed with: failed to retrieve openAPI spec, http error: ResponseCode: 503, Body: Error trying to reach service: 'x509: certificate relies on legacy Common Name field, use SANs or temporarily enable Common Name matching with GODEBUG=x509ignoreCN=0', Header: map[Content-Type:[text/plain; charset=utf-8] X-Content-Type-Options:[nosniff]]



 Error: tls: private key does not match public key



```bash
openssl req -new \
    -key firewalld.key \
    -subj "/CN=firewalld.default.svc" \
    -config <(cat /etc/pki/tls/openssl.cnf <(printf "[aa]\nsubjectAltName=DNS:firewalld, DNS:firewalld.default.svc, DNS:firewalld-certificate-authority, DNS:kubernetes.default.svc")) \
    -out firewalld.csr

openssl ca \
	-in firewalld.csr \
	-cert front-proxy-ca.crt \
	-keyfile front-proxy-ca.key \
	-out firewalld.crt \
	-days 3650 \
	-extensions aa \
	-extfile <(cat /etc/pki/tls/openssl.cnf  <(printf "[aa]\nsubjectAltName=DNS:firewalld, DNS:firewalld.default.svc, DNS:firewalld-certificate-authority, DNS:kubernetes.default.svc"))
```





github.com/flannel-io/flannel/subnet/kube/kube.go:403: watch of *v1.Node ended with: an error on the server ("unable to decode an event from the watch stream: context canceled") has prevented the request from succeeding



```bash
E0729 06:56:09.253632       1 main.go:330] Error registering network: failed to acquire lease: node "master-machine" pod cidr not assigned
I0729 06:56:09.253682       1 main.go:447] Stopping shutdownHandler...
W0729 06:56:09.253809       1 reflector.go:436] github.com/flannel-io/flannel/subnet/kube/kube.go:403: watch of *v1.Node ended with: an error on the server ("unable to decode an event from the watch stream: context canceled") has prevented the request from succeeding
```



[kube flannel cant get cidr although podcidr available on node](https://stackoverflow.com/questions/50833616/kube-flannel-cant-get-cidr-although-podcidr-available-on-node)



`mvcc: required revision has been compacted`

```log
kube-apiserver: W1120 17:18:20.199950   70454 watcher.go:207] watch chan error: etcdserver: mvcc: required revision has been compacted
```

这里是etcd返回的错误，被apiserver视为警告，在注释中有这么一句话

> If the context is "context.Background/TODO", returned "WatchChan" will not be closed and block until event is triggered, except when server returns a non-recoverable error (e.g. ErrCompacted).
>
> 如果这个上下文返回WatchChan将在下次事件被触发前不会被关闭或阻塞，除非服务器返回一个ErrCompacted （不可恢复）

对于etcd 对 Revision 有如下说明

etcd对每个kv的revision 都会保留一个压缩周期的值，例如每5分钟收集一次最新revision，当压缩周期达到时，将从历史记录中后去最后一个修订版本，例如为100，此时会压缩，压缩成功会重置计数器，并以最新的revision和新的历史记录进行开始，压缩失败将在5分钟后重试`--auto-compaction-retention=10` 是配置压缩周期的  每多少个小时键值存储运行定期压缩。 

如果最新的revision已被修订，etcd返回一个 `ErrCompacted`  表示已修订，此时表示为不可恢复状态

返回错误时表示这个watch被关闭，为了优雅的关闭chan，kubernetes会对这个watch错误进行返回，而 `ErrCompacted`  本质上不算错误

```go
wch := wc.watcher.client.Watch(wc.ctx, wc.key, opts...)
for wres := range wch {
    if wres.Err() != nil {
        err := wres.Err()
        // If there is an error on server (e.g. compaction), the channel will return it before closed.
        logWatchChannelErr(err)
        wc.sendError(err)
        return
    }
    ...
```

https://etcd.io/docs/v3.5/op-guide/maintenance/#auto-compaction

