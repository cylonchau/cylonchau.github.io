# istio安装


Istio用于提供统一方式来集成微服务的开放平台，管理[微服务之间的](https://istio.io/latest/docs/examples/microservices-istio/)[流量](https://istio.io/latest/docs/concepts/traffic-management/)，执行策略和汇总遥测数据。Istio的控制平面在基础集群管理平台（例如Kubernetes）上提供了一个抽象层。

Istio组成：



在Istio1.8中，istio由以下组件组成：[istio-component](https://istio.io/latest/docs/ops/deployment/architecture/)

istio服务网格分为数据平面和控制平面

- 数据平面：数据平面是由一组代理组成
  - envoy：Sidecar Proxy 每个微服务代理来处理入口/出口业务服务之间的集群中，并从外部服务的服务。代理形成一个*安全的微服务网格，*提供了丰富的功能集合

- 控制平面：管理与配置代理的流量规则。
  - istiod：istio的控制平面，提供了服务发现，配置和证书管理，包含如下组件：
    - **Pilot** ：负责运行时配置，（服务发现，智能路由）
    - **Citadel**：负责证书的颁发与轮替
    - **Galley**：负责配置的管理（验证，提取，分发等功能）

istio卸载

bookinfo卸载

```
kubectl delete -f samples/bookinfo/platform/kube/bookinfo.yaml
```

istio卸载

```
istioctl manifest generate|kubectl delete -f -
```

addons

```
kubectl delete -f samples/addons/prometheus.yaml
kubectl delete -f samples/addons/jaeger.yaml
kubectl delete -f samples/addons/kiali.yaml
kubectl delete -f samples/addons/grafana.yaml
```


