# Kubernetes Dashboard


基于web的UI前端，认证是由Kubernetes完成的。登陆dashboard的密码是k8s的账号和密码，和dashboard自身没有关系。dashboard自身不做认证。


```
kubectl patch svc kubernetes-dashboard -p&#39;{&#34;spec&#34;:{&#34;type&#34;:&#34;NodePort&#34;}}&#39;-n kube-system
```

如使用域名访问，CN一定要与域名保持一致。

```
(umask 077; openssl genrsa -out dashboard.key 2048)
openssl req -new -key dashboard.key -out dashboard.csr -subj &#34;/O=test/CN=dashboard&#34;
openssl req -in dashboard.csr -noout -text
openssl x509 -req -in dashboard.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out dashboard.crt -days 3650
```

[Certificate Attributes](https://docs.oracle.com/cd/E24191_01/common/tutorials/authz_cert_attributes.html)


要想穿透集群边界，从集群外访问集群内部某一服务或Pod上的容器的应用，有两种方式 nodePort、NodeBlanc 或ingress

```
kubectl create secret generic \
dashboard-cert \
-n kube-system \
--from-file=dashboard.crt=./dashboard.crt \
--from-file=dashboard.key=./dashboard.key
```

dashboard运行在Pod中时，当用户通过浏览器来进行登陆时，所提供的认证证书必须时serviceaccount，


```
kubectl create serviceaccount dashboard-admin -n kube-system
```

通过rolebindding吧对应的dashboard-admin和集群管理员建立起绑定关系，否则无法透过rbac的权限检查。

指明serviceaccount时，必须指明是哪个名称空间的的哪个账号，格式：&lt;font color=&#34;#f8070d&#34; size=3&gt;`namespace:serviceaccount`&lt;/font&gt;

```
kubectl create clusterrolebinding dashboard-cluster-admin --clusterrole=cluster-admin --serviceaccount=kube-system:dashboard-admin
```

使serviceaccount用户，能够有权限访问整个集群级别的资源。

查询dashboard-admin的secret，这个是自动生成的。

```
kubectl get secret $(kubectl get secret -n kube-system|grep dashboard-admin-token|awk &#39;{print $1}&#39;) -n kube-system -o jsonpath={.data.token}|base64 -d
```


kubectl config set-credentials 命令，用户的认证方式，既可以使用证书方式，也可以使用token认证。

在apply之前先将证书做成secret，apply操作会将其加载成为apply操作对外提供https服务时使用的证书。此操作作为serviceaccount认证是没有关系的，只不过是被dashboard用来做https证书的。如果不提供证书，dashboard会自动生成新的证书。

