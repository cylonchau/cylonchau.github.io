# Kubernetes安装错误


apiserver报错 没有kubeappserver用户

```
Failed at step USER spawning  No such process
```



```log
Mar  2 04:54:56 node02 systemd: controller-manager.service: main process exited, code=exited, status=1/FAILURE
Mar  2 04:54:56 node02 kube-controller-manager: Get http://127.0.0.1:8443/api/v1/namespaces/kube-system/configmaps/extension-apiserver-authentication: net/http: HTTP/1.x transport connection broken: malformed HTTP response &#34;\x15\x03\x01\x00\x02\x02&#34;
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
Switched to context &#34;system:controller-manager@k8s&#34;.
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
Nov 14 16:31:03 master01 kubelet: E1114 16:31:03.677280    8587 kubelet.go:2270] node &#34;master01&#34; not found
```



```
k8s.io/client-go/informers/factory.go:135: Failed to list *v1.CSIDriver: csidrivers.storage.k8s.io is forbidden: User &#34;system:anonymous&#34; cannot list resource &#34;csidrivers&#34; in API group &#34;storage.k8s.io&#34; at the cluster scope
```



```
380   19810 kubelet.go:2292] node &#34;master-machine&#34; not found
May 12 23:45:11 master-machine kubelet: E0512 23:45:11.415099   19810 kubelet.go:2292] node &#34;master-machine&#34; not found
May 12 23:45:11 master-machine kubelet: E0512 23:45:11.460097   19810 certificate_manager.go:434] Failed while requesting a signed certificate from the master: cannot create certificate signing request: certificatesigningrequests.certificates.k8s.io is forbidden: User &#34;system:anonymous&#34; cannot create resource &#34;certificatesigningrequests&#34; in API group &#34;certificates.k8s.io&#34; at the cluster scope
```



&gt; 问题：`&#34;master01&#34; is forbidden: User &#34;system:anonymous&#34; `
&gt;
&gt; 原因：用户未授权
&gt;
&gt; 解决  ：`kubectl create clusterrolebinding kube-apiserver:kubelet-apis --clusterrole=system:kubelet-api-admin --user kubernetes-master` &amp;&amp; `kubectl certificate approve`

```
master01 kubelet: E1114 17:16:33.581116    8559 kubelet.go:2270] node &#34;master01&#34; not found

failed to ensure node lease exists, will retry in 6.4s, error: leases.coordination.k8s.io &#34;master01&#34; is forbidden: User &#34;system:anonymous&#34; cannot get resource &#34;leases&#34; in API group &#34;coordination.k8s.io&#34; in the namespace &#34;kube-node-lease&#34;
```

&gt;问题1： flannela访问`dial tcp 10.96.0.1:443: connect: connection timed out` 
&gt;
&gt;问题2：`open /run/flannel/subnet.env: no such file or directory`
&gt;
&gt;问题3： `Container runtime network not ready: NetworkReady=false reason:NetworkPluginNotReady message:docker: network plugin is not ready: cni config uninitialized`
&gt;
&gt;解决：检查kube-proxy组件

```
Nov 14 17:30:04 master01 kubelet: E1114 17:30:04.097261    1160 kubelet.go:2190] Container runtime network not ready: NetworkReady=false reason:NetworkPluginNotReady message:docker: network plugin is not ready: cni config uninitialized
```

```
[root@master01 kubernetes]# kubectl logs kube-flannel-ds-dvsv7 -n kube-system
....
E1114 23:22:20.984238       1 main.go:243] Failed to create SubnetManager: error retrieving pod spec for &#39;kube-system/kube-flannel-ds-dvsv7&#39;: Get &#34;https://10.96.0.1:443/api/v1/namespaces/kube-system/pods/kube-flannel-ds-dvsv7&#34;: dial tcp 10.96.0.1:443: connect: connection timed out
[root@master01 kubernetes]# 
```

```
create pod sandbox: rpc error: code = unknown desc = failed to set up sandbox container : networkplugin cni failed to set up pod &#34;coredns-8686db44f5-tgpp4_kube-system&#34; network: open /run/flannel/subnet.env: no such file or directory
```

&gt; 问题：kubectl get csr 显示No Resources Found的解决记录
&gt;
&gt; 解决：&lt;font size=2&gt;需要先将 bootstrap token 文件中的 kubelet-bootstrap 用户赋予 system:node-bootstrapper 角色，然后 kubelet 才有权限创建认证请求&lt;/font&gt;



```
{&#34;log&#34;:&#34;E1204 15:10:26.155093       1 main.go:234] Failed to create SubnetManager: error retrieving pod spec for &#39;kube-system/kube-flannel-ds-9pmsp&#39;: Get \&#34;https://192.160.0.1:443/api/v1/namespaces/kube-system/pods/kube-flannel-ds-9pmsp\&#34;: x509: certificate is valid for 192.168.0.1, 10.0.0.5, not 192.160.0.1\n&#34;,&#34;stream&#34;:&#34;stderr&#34;,&#34;time&#34;:&#34;2021-12-04T15:10:26.156115275Z&#34;}
```





https://blog.51cto.com/juestnow/2439614

```log
runtime network not ready: NetworkReady=false reason:NetworkPluginNotReady message:docker: network plugin is not ready: cni config uninitialized
```



## 授予KUBE-APISERVER对KUBELET API的访问权限

kubelet启动启动时，`--kubeletconfig` 使用参数对应的文件是否存在 如果不存在`--bootstrap-kubeconfig`指定的kubeconfig文件向kube-apiserver发送CSR请求

kube-apiserver 收到 CSR 请求后，对其中的 token 进行认证，认证通过后将请求的用户设置为`system:bootstrap:&lt;Token ID&gt;`，组设置为 ，`system:bootstrappers`此操作称为`Bootstrap Token Auth`。

默认这个用户和组没有创建CSR的权限，kubelet没有启动，错误日志如下：

```
Unable to register node &#34;master-machine&#34; with API server: nodes is forbidden: User &#34;system:anonymous&#34; cannot create resource &#34;nodes&#34; in API group &#34;&#34; at the cluster scope
```

解决方法是：创建一个集群角色绑定，绑定一个组：`system:bootstrapper` 和一个clusterrole `system:node-bootstrapper`

```
$ kubectl create clusterrolebinding kubelet-bootstrap --clusterrole=system:node-bootstrapper --group=system:bootstrappers
```



kubectl create clusterrolebinding kubelet-bootstrap --clusterrole=system:node-bootstrapper --group=system:bootstrappers

system:serviceaccount:default:default



kubectl create clusterrolebinding firewalld-default --clusterrole=system:aggregate-to-admin --user=system:serviceaccount:default:default

system:aggregate-to-admin





Jun 22 16:53:10 master-machine kube-apiserver: E0622 16:53:10.620143   28763 controller.go:114] loading OpenAPI spec for &#34;v1.firewalld.fedoraproject.org&#34; failed with: failed to retrieve openAPI spec, http error: ResponseCode: 503, Body: Error trying to reach service: &#39;x509: certificate relies on legacy Common Name field, use SANs or temporarily enable Common Name matching with GODEBUG=x509ignoreCN=0&#39;, Header: map[Content-Type:[text/plain; charset=utf-8] X-Content-Type-Options:[nosniff]]



 Error: tls: private key does not match public key



```bash
openssl req -new \
    -key firewalld.key \
    -subj &#34;/CN=firewalld.default.svc&#34; \
    -config &lt;(cat /etc/pki/tls/openssl.cnf &lt;(printf &#34;[aa]\nsubjectAltName=DNS:firewalld, DNS:firewalld.default.svc, DNS:firewalld-certificate-authority, DNS:kubernetes.default.svc&#34;)) \
    -out firewalld.csr

openssl ca \
	-in firewalld.csr \
	-cert front-proxy-ca.crt \
	-keyfile front-proxy-ca.key \
	-out firewalld.crt \
	-days 3650 \
	-extensions aa \
	-extfile &lt;(cat /etc/pki/tls/openssl.cnf  &lt;(printf &#34;[aa]\nsubjectAltName=DNS:firewalld, DNS:firewalld.default.svc, DNS:firewalld-certificate-authority, DNS:kubernetes.default.svc&#34;))
```





github.com/flannel-io/flannel/subnet/kube/kube.go:403: watch of *v1.Node ended with: an error on the server (&#34;unable to decode an event from the watch stream: context canceled&#34;) has prevented the request from succeeding



```bash
E0729 06:56:09.253632       1 main.go:330] Error registering network: failed to acquire lease: node &#34;master-machine&#34; pod cidr not assigned
I0729 06:56:09.253682       1 main.go:447] Stopping shutdownHandler...
W0729 06:56:09.253809       1 reflector.go:436] github.com/flannel-io/flannel/subnet/kube/kube.go:403: watch of *v1.Node ended with: an error on the server (&#34;unable to decode an event from the watch stream: context canceled&#34;) has prevented the request from succeeding
```



[kube flannel cant get cidr although podcidr available on node](https://stackoverflow.com/questions/50833616/kube-flannel-cant-get-cidr-although-podcidr-available-on-node)




