# helm


## 什么是helm

helm是一个用于kubernetes包管理工具，可简化Kubernetes资源的部署和管理。

### helm 名词

- *chart*：类似于rpm包，包含Kubernetes资源所需要的必要信息。
- *repo*：chart仓库，类似于yum的仓库，chart仓库是一个简单的HTTP服务。
- *values*：提供了自定义信息用来覆盖模板中的默认值。
- *release* ：chart安装后的版本记录。

## 安装helm

`Helm client` 即 helm v3 可以在官方github 下载对应的二进制包。

添加源

```bash
helm repo add stable http://mirror.azure.cn/kubernetes/charts/
helm repo add incubator http://mirror.azure.cn/kubernetes/charts-incubator/
```

```bash
helm list 查看发布

helm remove 删除

helm repo add xxx url  添加仓库

helm upgrade 更新

helm rollback 回滚

```

--set覆盖chart默认values值


```
`helm install http://mirror.azure.cn/kubernetes/charts/jenkins-2.5.4.tgz --generate-name` 

helm install stable/jenkins --namespace kube-system   指定命名空间安装

helm install stable/jenkins --set mysqlRootPassword=123456  `--set` 优先级高于-f

helm install stable/jenkins --set persistence.enable=false

helm install stable/jenkins --set ports[0].containerPort=80,--set=ports[1].containerPort=81

helm install stable/jenkins --set name=value1\,value2

helm install -f test.yaml


```

show 查看chart值，可查看在线和离线

```
helm show values stable/jenkins | url | file  查看chart包values

helm search repo repo_name 搜索仓库内容
```

将chart下载到本地

```
helm fetch stable/minio
```

chart

Charts 是创建在特定目录下面的文件集合，然后可以将它们打包到一个版本化的存档中来部署。接下来我们就来看看使用 Helm 构建 charts 的一些基本方法。



`Chart.yaml`文件是chart必需的。包含了以下字段：

[apiVersion 版本的说明](https://github.com/helm/helm/issues/5907)

```yaml
apiVersion: chart API 版本 （必需）
name: chart名称 （必需）
version: 语义化2 版本（必需）
kubeVersion: 兼容Kubernetes版本的语义化版本（可选）
description: 一句话对这个项目的描述（可选）
type: chart类型 （可选）
keywords:
  - 关于项目的一组关键字（可选）
home: 项目home页面的URL （可选）
sources:
  - 项目源码的URL列表（可选）
dependencies: # chart 必要条件列表 （可选）
  - name: chart名称 (nginx)
    version: chart版本 (&#34;1.2.3&#34;)
    repository: 仓库URL (&#34;https://example.com/charts&#34;) 或别名 (&#34;@repo-name&#34;)
    condition: （可选） 解析为布尔值的yaml路径，用于启用/禁用chart (e.g. subchart1.enabled )
    tags: # （可选）
      - 用于一次启用/禁用 一组chart的tag
    enabled: （可选） 决定是否加载chart的布尔值
    import-values: # （可选）
      - ImportValue 保存源值到导入父键的映射。每项可以是字符串或者一对子/父列表项
    alias: （可选） chart中使用的别名。当你要多次添加相同的chart时会很有用
maintainers: # （可选）
  - name: 维护者名字 （每个维护者都需要）
    email: 维护者邮箱 （每个维护者可选）
    url: 维护者URL （每个维护者可选）
icon: 用做icon的SVG或PNG图片URL （可选）
appVersion: 包含的应用版本（可选）。不需要是语义化的
deprecated: 不被推荐的chart （可选，布尔值）
annotations:
  example: 按名称输入的批注列表 （可选）.
```



`--dry-run` 模拟安装

`--debug` 详细的输出

`--generate-name`：生成随机实例名

```
helm install --generate-name --dry-run --debug --set favoriteDrink=7up ./testchart
```

go template

`{{ .Values.favoriteDrink }}` 读取变量值

`quote` go template 函数，[引用字符串](https://helm.sh/docs/chart_template_guide/functions_and_pipelines/) ，给变量值加`&#34;&#34;`

`pipeline` 可将多个功能连接在一起

默认值 `chart.yaml`  `gender: {{ .values.gender|default &#34;zhangsan&#34; }}` values.yaml 中不能加default函数

流程控制

[if else](https://helm.sh/docs/chart_template_guide/control_structures/)

```
{{ if PIPELINE }}
  # Do something
{{ else if OTHER PIPELINE }}
  # Do something else
{{ else }}
  # Default case
{{ end }}
```

如果 pipeline的值被判定为如下的值则为false：

- a boolean false
- a numeric zero
- an empty string
- a `nil` (empty or null)
- an empty collection (`map`, `slice`, `tuple`, `dict`, `array`)

使用 `-` 摆脱新行`{{- if eq .Values.favorite.drink &#34;coffee&#34; }}` `-` 在前面表示删除前面的空行，在后面表示删除后面的空行，`{{-` 之间没有空格



`{{ indent 2 &#34;mug:true&#34; }}` 缩进，缩进的是文字内容不是yaml

`{{with .Values.xxx}} {{end}}` 提升作用于，`release: {{ $.Release.Name }}` 执行时将变量映射到根域，可以在作用域中使用
