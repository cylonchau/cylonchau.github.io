# jenkins多个slave遇到的坑


 ## jenkins配置如下

在Jenkins上添加了两个节点(Slave Node)，且为这两个节点设置了一个相同的标签"windows"。创建了一个新Job –  "test-windows"，选择的是”构建一个自由风格的软件项目”。并且为了使多个slave并行构建，我选择了"只允许绑定到这台机器的job”，在"Label Expression"中选择了"windows"。

![image-20221025231428874](https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/image-20221025231428874.png)

然而这种方式并不能实现多个slave并行操作。网上90%说的都不靠谱。

在我使用的过程中，使用了label 去管理多个 Slave，给一个项目的构建指定了这个 label，会发现这个项目的多次构建，都使用同一个 Slave，并没有使用 label 里的其它 Slave去构建。

查了很多资料才发现原来从 jenkins 的调度算法使用了一致性的哈希算法，jenkins根据添加的信息评测出优先级列表，选择优先级最高的Slave去构建，当最优slave不满足条件或者没有可用的 execut时，才会选用下一个slave。

查了很多资料发现构造多配置项目可以选择构建时的slave。这样可以实现多slave并行构建。

![image-20221025231449517](https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/image-20221025231449517.png)

multi configuration project比起构建自由风格的软件项目多个Configuration Matrix，在这里可以选择多个slave。这里选择lable的话，还是会使用默认算法从lable中选择最优slave进行构建。

![image-20221025231504755](https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/image-20221025231504755.png)

配置完成后再构建时，会同时在多个slave上进行并行构建

![image-20221025231536022](https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/image-20221025231536022.png)

禁止在master上运行job或和业务相关的操作

将 [executors] 设置为0

![image-20221025231520553](https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/image-20221025231520553.png)
