# 

https://kubebuilder.io/



[operator-sdk下载地址](https://github.com/operator-framework/operator-sdk) ，下载后放置PATH路径下

[operator-sdk document](https://sdk.operatorframework.io/)



新建代码

```
operator-sdk help
operator-sdk init help
operator-sdk init --domain chinamobile.com --owner seal

operator-sdk create help
operator-sdk create api --group app --version v1beta1 --kind End
```

创建完成后的文件结构

```bash
├── PROJECT
├── api
│   └── v1beta1
│       ├── end_types.go # 这个是需要定义的crd的类型
│       ├── groupversion_info.go # 这个是组版本信息
│       └── zz_generated.deepcopy.go # 这个自动生成的深拷贝
├── bin
│   └── manager
├── config
│   ├── certmanager
│   │   ├── certificate.yaml
│   │   ├── kustomization.yaml
│   │   └── kustomizeconfig.yaml
│   ├── crd
│   │   ├── kustomization.yaml
│   │   ├── kustomizeconfig.yaml
│   │   └── patches
│   │       ├── cainjection_in_ends.yaml
│   │       └── webhook_in_ends.yaml
│   ├── default
│   │   ├── kustomization.yaml
│   │   ├── manager_auth_proxy_patch.yaml
│   │   ├── manager_webhook_patch.yaml
│   │   └── webhookcainjection_patch.yaml
│   ├── manager
│   │   ├── kustomization.yaml
│   │   └── manager.yaml
│   ├── prometheus
│   │   ├── kustomization.yaml
│   │   └── monitor.yaml
│   ├── rbac
│   │   ├── auth_proxy_client_clusterrole.yaml
│   │   ├── auth_proxy_role.yaml
│   │   ├── auth_proxy_role_binding.yaml
│   │   ├── auth_proxy_service.yaml
│   │   ├── end_editor_role.yaml
│   │   ├── end_viewer_role.yaml
│   │   ├── kustomization.yaml
│   │   ├── leader_election_role.yaml
│   │   ├── leader_election_role_binding.yaml
│   │   └── role_binding.yaml
│   ├── samples
│   │   ├── app_v1beta1_end.yaml # 这是对应crd的yaml文件
│   │   └── kustomization.yaml
│   ├── scorecard
│   │   ├── bases
│   │   │   └── config.yaml
│   │   ├── kustomization.yaml
│   │   └── patches
│   │       ├── basic.config.yaml
│   │       └── olm.config.yaml
│   └── webhook
│       ├── kustomization.yaml
│       ├── kustomizeconfig.yaml
│       └── service.yaml
├── controllers
│   ├── end_controller.go # 实现的自己的业务逻辑
│   └── suite_test.go
├── go.mod
├── go.sum
├── hack
│   └── boilerplate.go.txt
└── main.go
```

实现一个此yaml的crd

```yaml
apiVersion: app.chinamobile.com/v1beta1
kind: End
metadata:
  name: end-sample
spec:
  size: 2
  image: sealloong/envoy_end
  ports:
    - port: 90
      targetPort: 90
      nodePort: 33000
```



```yaml
// EndSpec defines the desired state of End
type EndSpec struct {
	// INSERT ADDITIONAL SPEC FIELDS - desired state of cluster
	// Important: Run "make" to regenerate code after modifying this file

	// Foo is an example field of End. Edit End_types.go to remove/update
	// ex: Foo string `json:"foo,omitempty"`
	Size int32 `json:"size"`
	Image string `json:"image"`
	Ports []v1.ServicePort `json:"ports"`
	Resources v1.ResourceRequirements `json:"resources,omitempty"`
	Env []v1.EnvVar `json:"env,omitempty"`
}
```

修改为代码后，执行make
