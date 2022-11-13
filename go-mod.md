# go mod


自从Go官方推出 1.11 之后，增加新的依赖管理模块并且更加易于管理项目中所需要的模块。模块是存储在文件树中的 Go 包的集合，其根目录中包含 go.mod 文件。 go.mod 文件定义了模块的模块路径，它也是用于根目录的导入路径，以及它的依赖性要求。每个依赖性要求都被写为模块路径和特定语义版本。

从 Go 1.11 开始，Go 允许在 `$GOPATH/src` 外的任何目录下使用 go.mod 创建项目。在 `$GOPATH/src` 中，为了兼容性，Go 命令仍然在旧的 GOPATH 模式下运行。从 ==Go 1.13== 开始，模块模式将成为默认模式。

使用模块开发 Go 代码时出现的一系列常见操作：


* 创建一个新模块。
* 添加依赖项。
* 升级依赖项。
* 删除未使用的依赖项。

要使用go module,首先要设置 ==`GO111MODULE=on`== ,如果没设置，执行命令的时候会有提示。

==`GO111MODULE`== 的取值为 `off`, `on`, `or auto` (默认值，因此前面例子里需要注意2个重点)。

- **`off`**: `GOPATH mode`，查找vendor和GOPATH目录
- **`on`**：`module-aware mode`，使用 go module，忽略GOPATH目录
- **`auto`**：如果当前目录不在`$GOPATH` 并且 当前目录（或者父目录）下有go.mod文件，则使用`GO111MODULE`， 否则仍旧使用 `GOPATH mode`。

```sh
export GO111MODULE=on
export GOPROXY=https://goproxy.io ## 设置代理
```

### go mod 参数说明

| commond  | 说明                                                         |
| -------- | ------------------------------------------------------------ |
| download | download modules to local cache (下载依赖的module到本地cache)) |
| edit     | edit go.mod from tools or scripts (编辑go.mod文件)           |
| graph    | print module requirement graph (打印模块依赖图))             |
| init     | initialize new module in current directory (再当前文件夹下初始化一个新的module, 创建go.mod文件)) |
| tidy     | add missing and remove unused modules (增加丢失的module，去掉未用的module) |
| vendor   | make vendored copy of dependencies (将依赖复制到vendor下)    |
| verify   | verify dependencies have expected content (校验依赖)         |
| why      | explain why packages or modules are needed (解释为什么需要依赖) |

#### 新的项目

可以在`GOPATH`之外创建新的项目。

使用空目录创建`go.mod (module)`

```
go mod init {packagename}

go mod init test
```

#### `go get` 升级

- 运行 `go get -u` 将会升级到最新的次要版本或者修订版本(x.y.z, z是修订版本号， y是次要版本号)
- 运行 `go get -u=patch` 将会升级到最新的修订版本
- 运行 `go get package@version` 将会升级到指定的版本号version
- 运行`go get`如果有版本的更改，那么`go.mod`文件也会更改

#### 包管理

当我们使用`go build`，`go test`以及`go list`时，go会自动得更新go.mod文件，将依赖关系写入其中。

下载的包保存在`$GOPATH/`




#### 升级依赖项

查看使用到的依赖列表 `go list -m all`

```sh
root@lc-virtual-machine:~# go list -m all
chat
github.com/Knetic/govaluate v3.0.0&#43;incompatible
github.com/OwnLocal/goes v1.0.0
github.com/astaxie/beego v1.12.0
github.com/beego/goyaml2 v0.0.0-20130207012346-5545475820dd
github.com/beego/x2j v0.0.0-20131220205130-a0352aadc542
github.com/bradfitz/gomemcache v0.0.0-20180710155616-bc664df96737
github.com/casbin/casbin v1.7.0
github.com/cloudflare/golz4 v0.0.0-20150217214814-ef862a3cdc58
github.com/couchbase/go-couchbase v0.0.0-20181122212707-3e9b6e1258bb
github.com/couchbase/gomemcached v0.0.0-20181122193126-5125a94a666c
github.com/couchbase/goutils v0.0.0-20180530154633-e865a1461c8a
github.com/cupcake/rdb v0.0.0-20161107195141-43ba34106c76
github.com/edsrzf/mmap-go v0.0.0-20170320065105-0bce6a688712
github.com/elazarl/go-bindata-assetfs v1.0.0
github.com/garyburd/redigo v1.6.0
github.com/go-redis/redis v6.14.2&#43;incompatible
github.com/go-sql-driver/mysql v1.4.1
github.com/gogo/protobuf v1.1.1
github.com/golang/snappy v0.0.0-20180518054509-2e65f85255db
github.com/gomodule/redigo v2.0.0&#43;incompatible
github.com/lib/pq v1.0.0
github.com/mattn/go-sqlite3 v1.10.0
github.com/pelletier/go-toml v1.2.0
github.com/pkg/errors v0.8.0
github.com/shiena/ansicolor v0.0.0-20151119151921-a422bbe96644
github.com/siddontang/go v0.0.0-20180604090527-bdc77568d726
github.com/siddontang/ledisdb v0.0.0-20181029004158-becf5f38d373
github.com/siddontang/rdb v0.0.0-20150307021120-fc89ed2e418d
github.com/ssdb/gossdb v0.0.0-20180723034631-88f6b59b84ec
github.com/syndtr/goleveldb v0.0.0-20181127023241-353a9fca669c
github.com/wendal/errors v0.0.0-20130201093226-f66c77a7882b
golang.org/x/crypto v0.0.0-20181127143415-eb0de9b17e85
golang.org/x/net v0.0.0-20181114220301-adae6a3d119a
gopkg.in/check.v1 v0.0.0-20161208181325-20d25e280405
gopkg.in/yaml.v2 v2.2.1
```

&gt; **列出包的历史版本**

`go list -m -versions {package name}`

```sh
root@lc-virtual-machine:~# go list -m -versions github.com/astaxie/beego 
github.com/astaxie/beego v0.6.0 v0.7.0 v0.8.0 v0.9.0 v1.0.1 v1.2.0 v1.3.0 v1.4.0 v1.4.1 v1.4.2 v1.4.3 v1.5.0 v1.6.0 v1.6.1 v1.7.0 v1.7.1 v1.7.2 v1.8.0 v1.8.1 v1.8.2 v1.8.3 v1.9.0 v1.9.2 v1.10.0 v1.10.1 v1.11.0 v1.11.1 v1.12.0

```

&gt; **手动处理依赖关系**

`go mod tidy` 会自动清理掉不需要的依赖项，同时可以将依赖项更新到当前版本。

&gt; **切换包的版本**

```bash
go mod edit -require=&#34;github.com/astaxie/beego@v1.9.0&#34;
```

&gt; **清楚缓存**

```bash
go clean -modcache
```



### go mod replace

不过因为某些未知原因，并不是所有的包都能直接用go get获取到，这时我们就需要使用go modules的replace功能了。（当然大部分问题挂个梯子就能解决，但是我们也可以有其它选项）

```go.mod
replace (
	github.com/testcontainers/testcontainers-go =&gt; github.com/testcontainers/testcontainers-go v0.0.9
	golang.org/x/lint =&gt; github.com/golang/lint latest
)
```

修改后悔自动生成

```mod
replace (
	github.com/testcontainers/testcontainers-go =&gt; github.com/testcontainers/testcontainers-go v0.0.9
	golang.org/x/lint =&gt; github.com/golang/lint v0.0.0-20191125180803-fdd1cda4f05f
)
```


