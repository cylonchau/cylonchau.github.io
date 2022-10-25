# dodocker Registryker


## docker registry介绍

Registry用于保存docker镜像，包括镜像的层次结构和元数据，用户可自建`Registry`，也可使用官方的`Docker Hub`

分类

* Sponsor Registry：第三方的registry，供客户和Docker社区使用
* Mirror Registry：第三方的registry，只让客户使用
* Vendor Registry：由发布Docker镜像的供应商提供的registry
* Private Registry：通过设有防火墙和额外的安全层的私有实体提供的registry

一个docker Registry上拥有两种功能：

1. 提供镜像存储的仓库。
2. 提供用户获取镜像时的认证功能。
3. 同时提供当前服务器上所有可用镜像的搜索索引。

一个docker镜像仓库有仓库的名称，等同于yum的repostory。通常简称为repo。为了使的镜像和应用程序版本之间有意义上的关联关系。在docker一个仓库通常只存放一个应用程序的镜像。因此，这个仓库名就是应用程序名。通过给每个镜像额外添加一个组件叫<font color="#f8070d" size=3>`tag`</font>，来标识每一个镜像。通常镜像名称:标签<font color="#f8070d" size=3>`repo_name:tag`</font>才能唯一标识一个镜像。



为了可以快速创建registry，docker专门提供了一个程序包 <font color="#f8070d" size=3>`docker-distribution`</font> 。https://hub.docker.com/r/distribution/registry/ regustry自身运行在容器中，而容器的文件系统会随着容器生命周期终止而删除，因此需要给registry定义存储卷，使用网络存储。

&nbsp;在yum的extras仓库有一个<font color="#f8070d" size=3>`docker-registry`</font>的程序包。docker-distribution的主配置文件在 <font color="#f8070d" size=3>`/etc/docker-distribution/registry/config.yml`</font>，所有上传的镜像存放在<font color="#f8070d" size=3>`/var/lib/registry`</font> 。

```sh
root@node01 x86_64]# yum info docker-registry
Available Packages
Name        : docker-registry
Arch        : x86_64
Version     : 0.9.1
Release     : 7.el7
Size        : 123 k
Repo        : extras/7/x86_64
Summary     : Registry server for Docker
URL         : https://github.com/docker/docker-registry
License     : ASL 2.0
Description : Registry server for Docker (hosting/delivering of repositories and images).
```

### 配置docker registry访问

非 `docker hub` 必须给定registry的<font color="#f8070d" size=2>`地址`</font><font color="#f8070d" size=2>`端口`</font>，如果不是顶层仓库还要给定<font color="#f8070d" size=2>`用户名`</font>。
<font color="#f8070d" size=3>`docker push`</font> 默认基于https工作的，而服务器端使用的http，两者不兼容，需要标记为非加密、非安全的<font color="#f8070d" size=3>`docker registry`</font>。

```sh
[root@node01 ~]# docker push node02.com:5000/php
The push refers to repository [node02.com:5000/php]
Get https://node02.com:5000/v2/: http: server gave HTTP response to HTTPS client
```

编辑 <font color="#f8070d" size=3>`/etc/docker/daemon.json`</font> 添加 <font color="#f8070d" size=3>`insecure-registries`</font> ，并且名称一定要与仓库引用时使用的名称完全保持一致，多个以逗号分隔

```sh
{
    "insecure-registries": ["node02.com:5000"]
}
```

push的镜像存放在 <font color="#f8070d" size=3>`/var/lib/registry/`</font> 下，V2指的是registry的协议版本

```sh
$ ll /var/lib/registry/docker/registry/v2/
total 0
drwxr-xr-x 3 root root 20 Aug 28 23:31 blobs
drwxr-xr-x 3 root root 17 Aug 28 23:31 repositories
```

push时镜像会分层次，每一层都单独推送，单独存放。产生的镜像层次存放在 <font color="#f8070d" size=3>`php/_layers/sha256/`</font> ，真正存放的路径为 <font color="#f8070d" size=3>`/var/lib/registry/docker/registry/v2/blobs`</font> 下

```sh
$ ll /var/lib/registry/docker/registry/v2/repositories/php/_layers/sha256/
total 0
13bb1aa790b2a283bdeb26a9dd4afa0891e37252dd6f836e2bc8e1555903f7fd
256b176beaff7815db2a93ee2071621ae88f451bb1e198ca73010ed5bba79b65
3584183957db768fc11554dfd6b06ec41be02d7872cecb65aa5ba9f238c897e6
499f1709b835427d28bc4ddb1e7038a438f1a1272abfe5489d6c74cb69b51bec
6d33f059b806836d7e63f6f26f154b99a42abcc1d384da7569de593b8135f7fb
8158b516b87541f3641937087e8048977f48f9ced0bfaeb0bc007c1ea0d49b93
8fa12d754b796a48f42433fae8a8eee24b56679bba4e5648fa50b184622dd941
ca82288118de1328f65d428e6d2acc6a87ecf552ed5cc3698fde90cf76f3ebdb
d393fc3ffa9b40bfbddd978604e0d5249b0bb1a6e4953142d8b2c80fcc85bcb4
d9f1ee7bf8cab99a7362c98708edd26c2f622f13b8c74c3cafd6197442c20609
```

通过api获取中镜像与标签

```
$curl http://192.168.50.27:5000/v2/game/tags/list
{"name":"game","tags":["0123-151422","0124-162847","0124-164112","0125"]}
```

查看仓库中内容

```
$curl -XGET http://192.168.50.27:5000/v2/_catalog
{"repositories":["apiv1","game","php","tyapi"]}
```

## habor
· Project Harbor is an an open source trusted cloud native registry project that stores, signs, and scans content.
· Harbor extends the open source Docker Distribution by adding the functionalities usually required by users such as security, identity and management.
· Harbor supports advanced features such as user management, access control, activity monitoring, and replication between instances.


Feathers
·Multi-tenant content signing and validation
·Security and vulnerability analysis
·Audit logging
·Identity integration and role-based access control
·Image replication between instances
·Extensible API and graphical UI
·Internationalization (currently English and Chinese)

harbor托管在github上 https://github.com/goharbor/harbor
