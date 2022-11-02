# 容器的资源限制


默认情况下，容器没有任何资源限制，因此几乎耗尽docker主机之上，内核可分配给当前容器的所有资源。可以使用主机内核调度程序允许的尽可能多的给定资源。在此基础上Docker provides提供了控制容器可以使用多少内存，CPU或块IO的方法，设置docker run命令的运行时配置标志。

容器得以实现主要依赖于内核中的两个属性&lt;font color=&#34;#f8070d&#34; size=3&gt;`namespace cgroup`&lt;/font&gt;。其中许多功能都要求您的内核支持Linux功能。要检查支持，可以使用`docker info`命令。



Memory
OOME

在Linux主机上，如果内核检测到没有足够的内存来执行重要的系统功能，它会抛出OOME或`Out of Memory Exception`异常，并开始终止进程以释放内存资源。一旦发生OOME，任何进程都有可能被杀死，包括docker daemon自身在内。为此，Docker特地调整了docker daemon的OOM优选级，以免它被内核“正法”，但容器的优选级并未被调整。

工作逻辑为

在宿主机上跑有很多容器并包括系统级进程。系统级进程也包括docke daemon自身。当内核执行系统管理操作，如内核需要使用内存，发现可以内存已经为空，会启动评估操作，评估谁占用内存高。我们认为哪个资源占用内存高就该将其kill来释放内存空间。（需要注意的是占用内存高的进程也不一定被kill掉。A进程分配10G已使用5G，进程B分配1G已使用1G。A只使用50%内存，而B已经耗尽所有内存）。内核会提供这些进程进行评分，按照优先级逆序强制kill，直至可使用内存空间足够。此时内核就可以使用内存资源创建其他进程。

每一个进程被计算之后会有一个oom scores，得分越高就会被优先kill。得分是由内存申请分配空间等一系列复杂计算得知。当进程得分最高也不能被kill掉时，如docker daemon，此时需要调整优先级。每一个进程有一个oom.adj的参数，将优先级调整越低，计算的分数就越少。

在docker run时可以直接调整容器的OOM.adj参数。如果想限制容器能使用多少内存资源、或CPU资源，有专门的选项可以实现。非常重要的容器化应用需要在启动容器时调整其OOM.adj，还可以定义容器的策略，一旦被kill直接restart

[Limit a container&#39;s resources](https://docs.docker.com/config/containers/resource_constraints/#limit-a-containers-access-to-memory)

限制一个容器能使用多少内存资源或CPU资源docker有专门的选项来实现


`-m` 限制容器可用RAM空间。选项参数可以使用KB M G等作为接受单位使用。可单独使用。

`--memory-swap` 设置容器可用交换分区大小。使用swap允许容器在容器耗尽可用的所有RAM时将多余的内存需求写入磁盘。`--memory-swap`是一个修饰符标志，只有在设置了--memorys时才有意义。


`--memory-swap`

|\-\-memory-swap|\-\-memory  |功能|
|-|:--|-|
|正数S|正数 M   &amp;nbsp;&amp;nbsp;     |容器可用总空间为S，其中可用ram为M|
|0|正数|M相当于未设置swap（unset）|
|unset（未设置）|正数 M|若主机（Docker Host）启用了swap，则容器的可用swap为 `2*M` |
|-1|正数M|若主机（Docker Host）启用了swap，则容器可使用交换分区总空间大小为宿主机上的所有swap空间的swap资源|
|||注意：在容器内使用free命令可以看到的swap空间并不具有其所展现出的空间指示意义。|


--memory-swappiness

用来限定容器使用交换分区的倾向性。

--memory-reservation

预留的内存空间

--oom-kill-disable

禁止oom被kill掉

默认情况下，每个容器对主机CPU周期的访问权限是不受限制的。可以设置各种约束来限制给定容器访问主机的CPU周期。大多数用户使用和配置默认CFS调度程序。在Docker 1.13及更高版本中，还可以配置实时调度程序。
[CPU Limit a container&#39;s resources](https://docs.docker.com/config/containers/resource_constraints/#cpu)

内核中进程管理子系统当中最重要的组件为进程角度器scheduler，非实时优先级,有效范围为100-139[-20,19]。因此每个进程的默认优先级为120。实时优先级0-99。调度100-139之间的进程有个非常重要的调度器CFS scheduler（完全公平调度器），公平调度每一个进程在需要执行时，去分配scores到这个进程上。

在各容器之间分配CPU资源选项：

|参数|说明|
|-|-|
|--cpu-shares |限制CPU使用个数的参数按比例切分当前系统上可用cpu资源。&lt;br&gt;例如：当前系统上运行2各容器，第一个为1024，第二个为512。这两个容器都尽可能多个使用CPU，会将CPU资源分3份，1024占2份，第二个容器占1份。可随时按比例调整CPU资源。|
|--cups |指定容器可以使用的可用CPU资源量。例如，如果主机有两个CPU并且已设置--cpus=&#34;1.5&#34;，则容器最多保证1.5个CPU。例如：4核CPU，4个使用总量为1.5而不是0使用100%，1使用50%。|
|--cpuset-cpus|限制CPU使用范围的参数。限制容器可以使用的特定CPU或核心。当有多个CPU，则容器可以使用的以逗号分隔的列表或连字符分隔的CPU范围。第一个CPU编号为0.有效值可能是0-3 `1,3`使用第二个和第四个CPU。|

docker pull lorel/docker-stress-ng
