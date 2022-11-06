# k8s开发环境准备


## 下载源码

根据kubernetes github 方式可以 

```
mkdir -p $GOPATH/src/k8s.io
cd $GOPATH/src/k8s.io
git clone https://github.com/kubernetes/kubernetes
cd kubernetes
make
```

如果有需要可以切换到对应的版本进行学习或者修改，一般kubernetes版本为对应tag

```
git fetch origin [远程tag名]
git checkout  [远程tag名]
git branch
```

## 配置goland

kubernetes本身是支持gomod的，但源码这里提供了所有的依赖在`staging/src/k8s.io/` 目录下，可以将此目录内的文件复制到 `vendor`下。

对于 `k8s.io/kubernetes/pkg/` 发红的（找不到依赖的），可以将手动创建一个目录在 `vendor/k8s.io/kubernetes/` 将克隆下来的根目录 `pkg` 复制到刚才的目录下。

 goland中，此时不推荐使用go mod模式了，这里goland一定要配置GOPATH的模式。对应的GOPATH加入 `{project}/vender`即可。 这里可以添加到 goland中 `project GOPATH`里。

![image-20211116222531919](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20211116222531919.png)
