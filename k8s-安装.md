# 





```
Dec 24 00:44:35 node01 etcd: rejected connection from "10.0.0.16:55340" (error "remote error: tls: bad certificate", ServerName "node01
")                                                                                                                                     
Dec 24 00:44:36 node01 etcd: rejected connection from "10.0.0.17:39446" (error "remote error: tls: bad certificate", ServerName "node01
")
Dec 24 00:44:36 node01 etcd: rejected connection from "10.0.0.17:39450" (error "remote error: tls: bad certificate", ServerName "node01
")
Dec 24 00:44:36 node01 etcd: rejected connection from "10.0.0.16:55342" (error "remote error: tls: bad certificate", ServerName "node01
")
Dec 24 00:44:36 node01 etcd: rejected connection from "10.0.0.16:55346" (error "remote error: tls: bad certificate", ServerName "node01
")
```

解决方法： 
```
rm -fr /var/lib/etcd/k8s.etcd/*
```
重新生成整数。填写名字是k8s.test 修改每个主机主机名为   xxx.k8s.test 在hosts文件中解析





apiserver报错, apiserver启动，需要连上etcd来创建数据

Dec 24 18:08:15 node01 kube-apiserver: I1224 18:08:15.654540    2412 plugins.go:161] Loaded 6 validating admission controller(s) successfully in the following order: LimitRanger,ServiceAccount,Priority,PersistentVolumeClaimResize,ValidatingAdmissionWebhook,ResourceQuota.
Dec 24 18:08:15 node01 kube-apiserver: I1224 18:08:15.657518    2412 plugins.go:158] Loaded 8 mutating admission controller(s) successfully in the following order: NamespaceLifecycle,LimitRanger,ServiceAccount,NodeRestriction,Priority,DefaultTolerationSeconds,DefaultStorageClass,MutatingAdmissionWebhook.
Dec 24 18:08:15 node01 kube-apiserver: I1224 18:08:15.658434    2412 plugins.go:161] Loaded 6 validating admission controller(s) successfully in the following order: LimitRanger,ServiceAccount,Priority,PersistentVolumeClaimResize,ValidatingAdmissionWebhook,ResourceQuota.
Dec 24 18:08:35 node01 kube-apiserver: F1224 18:08:35.661432    2412 storage_decorator.go:57] Unable to create storage backend: config (&{ /registry [https://node01.k8s.test:2379 https://node02.k8s.test:2379 https://node03.k8s.test:2379] /etc/kubernetes/pki/apiserver-etcd-client.key /etc/kubernetes/pki/apiserver-etcd-client.crt /etc/etcd/pki/ca.crt true 0xc0003a8870 <nil> 5m0s 1m0s}), err (context deadline exceeded)
Dec 24 18:08:35 node01 systemd: kube-apiserver.service: main process exited, code=exited, status=255/n/a
Dec 24 18:08:35 node01 systemd: Failed to start Kubernetes API Server.
Dec 24 18:08:35 node01 systemd: Unit kube-apiserver.service entered failed state.
Dec 24 18:08:35 node01 systemd: kube-apiserver.service failed.
Dec 24 18:08:35 node01 systemd: kube-apiserver.service holdoff time over, scheduling restart.
Dec 24 18:08:35 node01 systemd: Starting Kubernetes API Server...




Dec 25 11:39:01 node01 etcd: listening for peers on https://10.0.0.15:2380
Dec 25 11:39:01 node01 etcd: listening for client requests on 10.0.0.15:2379
Dec 25 11:39:01 node01 etcd: couldn't find local name "k8s-etc" in the initial cluster configuration
Dec 25 11:39:01 node01 systemd: etcd.service: main process exited, code=exited, status=1/FAILURE
Dec 25 11:39:01 node01 systemd: Failed to start Etcd Server.
Dec 25 11:39:01 node01 systemd: Unit etcd.service entered failed state.


[Releases · containernetworking/plugins · GitHub](https://github.com/containernetworking/plugins/releases)



failed to run Kubelet: cannot create certificate signing request: certificatesigningrequests.certificates.k8s.io is forbidden: User

```bash
echo "export KUBECONFIG=/etc/kubernetes/auth/admin.conf" >> ~/.bash_profile 
. ~/.bash_profile 
```

 



kubectl create clusterrolebinding kubelet-bootstrap --clusterrole=system:node-bootstrapper --group=system:bootstrappers

[阿里云手动搭建k8s搭建中遇到的问题解决（持续更新） - 大JAVA解决方案 - CSDN博客](https://blog.csdn.net/wangshuminjava/article/details/81218367)


```sh
[root@node01 ~]# kubectl get  csr
NAME                                                   AGE     REQUESTOR             CONDITION
node-csr-a-MREQ1IybB0U5M8RP5FasSjckQOZiCoCYlf8ipDwx8   5m11s   system:bootstrapper   Pending
```


```sh
cat > /etc/sysconfig/modules/ipvs.modules << EOF
#!/bin/bash
ipvs_mods_dir="/usr/lib/modules/$(uname -r)/kernel/net/netfilter/ipvs"
for i in \$(ls \$ipvs_mods_dir | grep -o "^[^.]*"); do
    /sbin/modinfo -F filename \$i >/dev/null 
    if [ \$? -eq 0 ]; then
    /sbin/modprobe -- \$i 
    fi 
done
EOF
```


Get https://k8s.gcr.io/v2/: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)


[国内环境Kubernetes v1.12.1的安装与配置 - 耕耘实录 - CSDN博客](https://blog.csdn.net/solaraceboy/article/details/83308339)



```yaml
cat > /etc/sysconfig/modules/ipvs.modules <<EOF
#!/bin/bash
ipvs_modules="ip_vs ip_vs_lc ip_vs_wlc ip_vs_rr ip_vs_wrr ip_vs_lblc ip_vs_lblcr ip_vs_dh ip_vs_sh ip_vs_fo ip_vs_nq ip_vs_sed ip_vs_ftp nf_conntrack_ipv4"
for kernel_module in \${ipvs_modules}; do
    /sbin/modinfo -F filename \${kernel_module} > /dev/null 2>&1
    if [ $? -eq 0 ]; then
        /sbin/modprobe \${kernel_module}
    fi
done
EOF
chmod 755 /etc/sysconfig/modules/ipvs.modules && bash /etc/sysconfig/modules/ipvs.modules && lsmod | grep ip_vs
```





coredns部署

```sh
mkdir coredns && cd coredns
wget https://raw.githubusercontent.com/coredns/deployment/master/kubernetes/coredns.yaml.sed
wget https://raw.githubusercontent.com/coredns/deployment/master/kubernetes/deploy.sh
bash deploy.sh -i 10.96.0.10 -r "10.96.0.0/12" -s -t coredns.yaml.sed |kubectl apply -f - 
```


```sh
[root@node01 coredns]# bash deploy.sh -i 10.96.0.10 -r "10.96.0.0/12" -s -t coredns.yaml.sed |kubectl apply -f - 
serviceaccount/coredns created
clusterrole.rbac.authorization.k8s.io/system:coredns created
clusterrolebinding.rbac.authorization.k8s.io/system:coredns created
configmap/coredns created
deployment.extensions/coredns created
service/kube-dns created
```

```sh
[root@node01 coredns]# kubectl get pods -n kube-system
NAME                          READY   STATUS    RESTARTS   AGE
coredns-7748f7f6df-h8tbp      1/1     Running   0          109s
coredns-7748f7f6df-xqqks      1/1     Running   0          109s
kube-flannel-ds-amd64-24d25   1/1     Running   0          17h
kube-flannel-ds-amd64-lvnp8   1/1     Running   1          7d21h

```

[k8s集群配置使用coredns - 简书](https://www.jianshu.com/p/b1cc739fb07f)





```
etcdctl \
--key-file=/etc/k8s/etcd-client.key \
--cert-file=/etc/k8s/etcd-client.crt \
--ca-file=/etc/k8s/ca.crt  \
--endpoint="https://test.k8s.node02:2379" cluster-health
```





```
[Unit]
Description=Kubernetes API Server
Documentation=https://github.com/GoogleCloudPlatform/kubernetes
After=network.target
After=etcd.service

[Service]
EnvironmentFile=-/etc/kubernetes/config
EnvironmentFile=-/etc/kubernetes/apiserver
User=kube
ExecStart=/usr/local/kubernetes/server/bin/kube-apiserver \
	    $KUBE_LOGTOSTDERR \
	    $KUBE_LOG_LEVEL \
	    $KUBE_ETCD_SERVERS \
	    $KUBE_API_ADDRESS \
	    $KUBE_API_PORT \
	    $KUBELET_PORT \
	    $KUBE_ALLOW_PRIV \
	    $KUBE_SERVICE_ADDRESSES \
	    $KUBE_ADMISSION_CONTROL \
	    $KUBE_API_ARGS
Restart=on-failure
Type=notify
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
```







1.15 新特性  https://zhuanlan.zhihu.com/p/70005875 

Container runtime network not ready: NetworkReady=false reason:NetworkPluginNotReady message:docker: network plugin is not ready: cni config uninitialized
Nov 14 18:29:48 master01 kubelet: E1114 18:29:48.822202    1160 aws_credentials.go:77] while getting AWS credentials NoCredentialProviders: no valid providers in chain. Deprecated.

Container runtime network not ready: NetworkReady=false reason:NetworkPluginNotReady message:docker: network plugin is not ready: cni config uninitialized





pp4_kube-system(9bcb948c-5fdf-4b27-a559-544cf7fdfec3)" with CreatePodSandboxError: "CreatePodSandbox for pod \"coredns-8686db44f5-tgpp4_kube-system(9bcb948c-5fdf-4b27-a559-544cf7fdfec3)\" failed: rpc error: code = Unknown desc = failed to set up sandbox container \"bca3889df0018ad206cddd4dd6a695a55fb45d08db4706276f251bf6e4a2a65a\" network for pod \"coredns-8686db44f5-tgpp4\": networkPlugin cni failed to set up pod \"coredns-8686db44f5-tgpp4_kube-system\" network: open /run/flannel/subnet.env: no such file or directory"







```
2.Container runtime network not ready: NetworkReady=false reason:NetworkPluginNotReady message:docker: network plugin is not ready: cni config uninitialized，由于没有插件cni

解决：修改kubelet.conf配置文件去掉相关配置参数–network-plugin=cni，重启服务即可或者下在安装cni插件
```


