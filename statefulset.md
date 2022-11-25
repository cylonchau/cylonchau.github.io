# Understand Kubernetes - StatefulSet有状态应用副本集


Kubernetes从1.3开始支持PetSet宠物集，1.5更名为StatefulSet。在Kubernetes之上，StatefulSet一般主要用于管理具有以下特性的应用程序

- 每一个Pod、组件，稳定且唯一的网络标识符。
- 稳定且持久的存储。
- 有序、平滑的部署和扩展。
- 有序、平滑的终止和删除。
- 有序的滚动更新。

一般典型的StatefulSet有三个组件组成

- headless service
- StatefulSet 控制器
- volumeClaimTemplate 存储卷申请模板，在创建每一Pod时，会自动生成一个PVC，请求绑定一个PV达到有自己专用的存储卷

在使用deployment控制器时，每一个pod的名称是随机字符串，并且为无序的。但是在StatefulSet当中要求必须为有序的（r1~r8），启动时顺序 （r1~r8）；终止时逆序 （r8~r1）。每一个Pod、每一节点都不能随意被取代。Pod名称是作为识别pod唯一性的标识符，这个标识符必须稳定、持久、有效的。

如何确定Pod标识符稳定、持久、有效呢？此时需要**==headless service==**来确保解析的名称是直达后端Pod IP地址的，并确保给每一个Pod唯一的名称。

大多数有状态副本集都会用到有状态持久存储。对分布式系统来讲，最大的特点是数据是不一样的。所以每个Pod应该具有自己专用的存储，一定不能共享给其他节点。在基于模板创建的Pod，在模板中创建的存储卷对所有Pos都是一样的，意味着Pod都访问同一个存储卷。故StatefulSet基于Pod模板创建存储卷是不适用的。

任何一个StatefulSet控制器，都应该有三部分组成

- service，service必须为无头service ==`clusterIP: none`==
- StatefulSet控制器  `kind: StatefulSet`
- 

在创建StatefulSet `kubectl explain sts`

sts

- spec
  - replicas 有几个副本
  - selector 那些Pod是由自己管理的
  - **==serviceName==** ***-required-*** serviceName必须关联至某个无头服务之上，基于此无头服务才能给每一个Pod分配唯一的、持久的、固定的标识符。
  - template 定义Pod模板，对应的存储卷应为Pvc类型
  - volumeClaimTemplates 会自动完成两个功能： 为每一个Pod定义volume；在Pod所在名称空间中自动穿件PVC
