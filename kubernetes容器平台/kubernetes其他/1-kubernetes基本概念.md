## 一、Kubernetes介绍

Kubernetes 是谷歌以 Borg 为前身，基于谷歌 15年生产环境经验的基础上开源的一个项目，Kubernetes致力于提供跨主机集群的自动部署、扩展、高可用以及运行应用程序容器的平台。

### 有了Docker 为甚么还需要使用Kubernetes

在生产环境中，宿主机宕机 Docker 容器是无法自动恢复的，而在 Kubernetes 中，是有多个 Node 节点的，Master 在检测到某一台 Node 节点无法通信后，会自动把容器迁移到其他的 Node 节点继续提供服务

在真正的业务场景中一般会部署大量的业务容器，而直接使用裸容器的方式来管理容器环境是比较吃力的，但是也有一些其他工具提供了Docker 单机编排的功能，如：docker-compose；并且在某些程序需要使用多个副本来实现高可用或者增加负载节点时 Docker 也无法直接提供支持，而 Kubernetes 可以使用控制器来实现容器的多个副本以及一些更高级的功能；并且在某些业务容器运行过程中会出现一些假死状态，需要请求健康检查接口来判断容器是否存活的场景时，Docker 也无法提供支持而 Kubernetes中可以直接使用探针进行状态检测。

在生产环境对容器进行扩容、回滚、更新等不够灵活，Kuberetes可以在资源控制其中直接，Kuberetes会自动根据我们的调整来对容器进行扩缩容等操作

## 二、Kubernetes集群架构

![image-20230825165156959](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825165156959.png)

### k8s集群架构说明

Kubernetes 分为 Master 节点和 Node节点，是一个主从架构，Master节点是整个集群的管理控制节点，生产环境中是不建议运行任何应用程序，否则会极大的增加 Master 节点的负载从而降低性能，生产环境中建议部署多个 Master节点，节点数量根据 Node 节点规模进行适当的调整，在使用一个代理用来代理多个 Master 的流量请求；Node节点是 Kubernetes 集群中运行应用程序的节点，相当于计算节点。

- Master节点：整个集群的控制管理中心，负责容器的调度，也是整个集群访问管理的入口；
- Node节点：整个集群的计算节点，所有的容器都运行在该节点；
- etcd：一个键值对数据库，存储集群的信息，一般生产环境中建议部署三个以上节点（奇数个），由于 Kubernetes 集群中所有数据都存储在 etcd中，所以 Kubernetes 所有的组件都是无状态服务，所以备份 etcd数据就相当于备份整个 Kubernetes 集群；
- Load Balancer：表示代理，主要作用是用来代理 Master 节点，可以选用物理的代理如：F5，或者软件的HAProxy + Keepalived；

## 三、Master节点详解

Master 节点是整个集群控制管理中心，有三个 Kubernetes 组件，分别是：kube-apiserver、kube-controller-manager、kube-scheduler

### 3.1、kube-apiserver

官方文档：https://kubernetes.io/zh/docs/reference/command-line-tools-reference/kube-apiserver/

API Server 是整个集群的控制中枢，提供集群中各个模块之间的数据交换，并将集群状态和信息存储到分布式键值（key - value）存储系统 etcd 集群中。同时它也是集群管理、资源配额、提供完备的集群安全机制的入口，为集群各类资源对象提供增删改查以及 Watch 的 REST API 接口，是整个系统的数据总线和数据中心；API Server 设计上考虑了水平伸缩，也就是说，它可以通过部署多个实例进行伸缩，你可以运行多个 kube-apiserver 的多个实例，并在这些实例之间平衡流量。

### 3.2 kube-controller-manager

官方文档：https://kubernetes.io/zh/docs/reference/command-line-tools-reference/kube-controller-manager/

Controller Manager 是集群的状态管理器，保证 pod或其他资源达到期望值，也是需要和 API Server进行通信，在需要的时候创建、更新或删除它所管理的资源；

Controller Manager作为集群内部的管理控制中心，非安全默认端口 10252，负责集群内的 Node、Pod 副本、Endpoint（服务端点）、NameSpace（命名空间）、ServiceAccount（服务账号）、ResourceQuota（资源定额）的管理，当某个 Node 意外宕机时，Controller Manager 会及时发现并执行自动化修复流程，确保集群始终处于预期的工作状态。

目前，kubernetes自带的控制器包括副本控制器、节点控制器、命名空间控制器和服务器账号控制器等

- 节点控制器（Node Controller）：负责在节点出现故障时进行通知和响应；
- 任务控制器（Job Controller）：监测代表一次性任务的 Job对象，然后创建 Pods来行这些任务直至完成；
- 端点控制器（Endpoints Controller）：填充端点（Endpoints）对象，即加入Service 于 Pod；
- 服务账号和令牌控制器（Service Account & Token Controller）：为新的命名空间创建默认账号和 API访问令牌；

### 3.3、kube-scheduler

官方文档：https://kubernetes.io/zh/docs/reference/command-line-tools-reference/kube-scheduler/

Scheduler是整个集群的调度中心，主要通过调度算法将 Pod 分配到最佳的 Node节点，它通过 API Server 监听所有的 Pod的状态，一旦发现新的未被调度到任何的 Node节点的 Pod（PodSpec.NodeName为空）就会根据一系列策略选择最佳节点进行调度；

Scheduler在整个系统中起到“承上启下”的作用，承上：负责接收 Controller Manager 创建的新的 Pod，为其选择一个合适的 Node；启下：Node上的 kubelet 接管 Pod的生命周期；

调度器通过 Kubernetes 的监测（Watch）机制来发现集群中新创建且尚未被调度到 Node 上的 Pod；调度器会将发现的每一个未调度的 Pod 调度到一个合适的 Node 上来运行，调度器会依据上下文的调度原则来做出调度选择；

Kubernetes 调度器是一个控制面进程，负责将 Pod 指派到节点上，调度器基于约束和可用资源为调度队列中每个 Pod 确定其合法放置的节点。调度器之后对所有合法的节点进行排序，将 Pod 绑定到一个合适的节点，在同一个集群中可以使用多个不同的调度器；kube-scheduler是其参考实现；

#### 1.调度算法

Scheduler通过调度算法为待调度 Pod 列表的每个 Pod 从可用 Node 列表中选择一个最合适的 Node，并将信息写入到 etcd 中；

Node 节点上的 kubelet 通过 API Server 监听到 Scheduler 产生的 Pod 绑定信息，然后获取对应的 Pod 清单，下载 Image，并启动容器；

#### 2.调度策略

优选策略，默认为1

- 1.LeastRequestedPriority：优先从备选节点列表中选择资源消耗最小的节点（CPU+内存）；
- 2.CalculateNodeLabelPriority：优先选择含有指定Label的节点；
- 3.BalancedResourceAllocation：优先从备选节点列中选择各项资源使用率最均衡的节点；

#### 3.调度原理

调度器通过 Kubernetes 的监测（Watch）机制来发现集群中新创建且尚未被调度到 Node 上的 Pod，调度器会将发现的每一个未调度的 Pod 调度到一个合适的 Node 节点上来运行；调度器会依据下文的调度原则来做出调度选择，对每一个新创建的 Pod 或者是未被调度的 Pod， kube-scheduler 会选择一个最优的 Node 去运行这个 Pod；然而，Pod 内的每一个容器对资源都有不同的需求，而且 Pod 本身也有不同的资源需求，因此，Pod 在被调度到 Node 上之前，根据这些特定的资源调度需求，需要对集群中的 Node 进行一次过滤，在一个集群中，满足一个 Pod 调度请求的所有 Node 被称之为**可调度节点**；如果没有任何一个 Node 能满足 Pod 的资源请求，那么这个 Pod 将一直停留在**未调度状态**，直到调度器能够找到合适的 Node；

调度器先在集群中找到一个 Pod 的所有可调度节点，然后根据一系列函数对这些可调度节点**打分**，选出其中得分最高的 Node 来运行 Pod，之后调度器将这个调度决定通知给 kube-apiserver，这个过程叫做**绑定**；

在做调度决定时需要考虑的因素包括：单独和整体的资源请求、硬件/软件/策略限制、亲和以及反亲和要求、数据局域性、负载间的干扰等等；

**kube-scheduer给一个 pod 做调度选择包含两个步骤**

1. 过滤；
2. 打分

过滤阶段会将所有满足 Pod 调度需求的 Node 选出来，例如： PodFitsResources 过滤函数会检查候选 Node 的可用资源能否满足 Pod 的资源请求，在过滤之后，得出一个 Node 列表，里边包含了所有可调度节点；通常情况下，这个 Node 列表包含不止一个 Node；如果这个列表是空的，代表这个 Pod 不可调度；

在打分阶段，调度器会为 Pod 从所有可调度节点中选取一个最合适的 Node，根据当前启用的打分规则，调度器会给每一个可调度节点进行打分；

最后，kube-scheduler 会将 Pod 调度到得分最高的 Node 上，如果存在多个得分最高的 Node，kube-scheduler 会从中随机选取一个；

**kube-scheduler支持一下两种方式配置调度器的过滤和打分行为**

1. 调度策略允许配置过滤的**断言（Predicates）**和**打分的优先级（Priorities）**；
2. 调度配置允许配置实现不同调度阶段的插件，包括：**QueueSort、Filter、Score、Bind、Reserve、Permit**等等。也可以配置 kube-scheduler 运行不同的配置文件；

## 四、Node节点详解

Node 节点在整个集群中是运行所有容器的节点，主要有三个组件分别是：**kubelet、kube-proxy、runtime（容器运行时）**。

### 4.1、kubelet

官方文档：https://kubernetes.io/zh/docs/reference/command-line-tools-reference/kubelet/

负责监听节点上 Pod 的状态，同时负责上报节点和节点上面的 Pod 的状态，负责与 Master 节点通信，并管理节点上面的 Pod；

在 Kubernetes 集群中，每个 Node（又称 Minion）上都会启动一个 kubelet 服务进程。该进程用来处理 Master 节点下发到本节点的任务，管理 Pod 及 Pod 中的容器；每个 kubelt 进程都会在 API Server 上注册节点自身的信息，定期向 Master 汇报节点资源的使用情况，并通过 cAdvisor（顾问）监控容器和节点资源；可以把 kubelet 理解成 Server/Agent 架构中的 Agent，kubelet 是 Node 节点上的 Pod 管家；

一个在集群中每个 Node 节点上运行的代理，它保证容器（Containers）都运行在 Pod 中，kubelet 接收一组通过各类机制提供给它的 PodSpecs，确保这些 PodSpecs 中描述的容器处于运行状态且健康。kubelet 不会管理不是由 kubernetes 创建的容器；

### 4.2、kube-proxy

官方文档：https://kubernetes.io/zh/docs/reference/command-line-tools-reference/kube-proxy/

为了支持集群的水平扩展和高可用性，Kubernetes 抽象出了 Service 的概念，Service 是对一组 Pod 的抽象，它会根据访问策略（负载均衡策略）来访问这组 Pod；

kubernetes 在创建服务时会为服务分配一个虚拟 IP 地址，客户端通过访问这个虚拟 IP 地址来访问服务，服务则负责将请求转发到后端的 Pod 上，这其实就是一个反向代理，但与普通的反向代理又有一些不同：它的 IP 地址是虚拟的，若想从外面访问，则还需要一些技巧，它的部署和启停是有 Kubernetes 统一自动管理的。Service 只是一个概念，而真正将 Service 的作用落实的是它背后的 kube-proxy 服务进程；

在 Kubernetes 集群的每个 Node 上都会运行一个 kube-proxy 服务进程，可以把这个进程看做 Service 的透明代理兼负载均衡器，其核心功能是将某个 Service 的访问请求转发到后端的多个 Pod 实例上；

kube-proxy的三种工作模式：userspace（用户空间代理，已经淘汰）、iptables、ipvs

查看 kube-proxy 工作模式： 

```bash
[root@k8s-master-1 ~]# curl http://10.202.43.46:10249/proxyMode
ipvs
```

#### 4.2.1、kube-proxy工作模式详解

**三种工作模式**

```bash
userspace：kubernetes v1.2 及以后就已经淘汰
iptables：默认方式，v1.1开始支持，v1.2开始为默认模式
ipvs：v1.9引入，v1.11正式支持，需要安装 ipvsadm、ipset工具包和加载 ip_vs 内核模块
```

**iptables**

从 1.2 版本开始，Kubernetes将 iptables 作为 kube-proxy 的默认模式，其工作原理如下图所示，iptables 模式下的 kube-proxy 进程不在起到数据层面的 proxy 的作用，Client 向 Service 的请求流量通过 iptables 的 NAT 机制直接发送到目标 Pod，不经过 kube-proxy 进程的转发， kube-proxy进程只承担了控制层面的功能，即通过 API Server 的 Watch 接口实时追踪 Service 与 Endpoint 的变更信息，并更新 Node 节点上相应的 iptables 规则。

kube-proxy 监听 Kubernetes Master 增加和删除 Service 以及 Endpoint 的消息，对于每一个 Service，kube-proxy 创建相应的 iptables 规则，并将发送到 Service Cluster IP 的流量转发到 Service 后端提供服务的 Pod 的相应端口上。

注：虽然可以通过 Service 的 Cluster IP 和服务端口访问到后端 Pod 提供的服务，但该 Cluster IP 是 ping 不通的，其原因是 Cluster IP 只是 iptables 中的规则，并不对应到任何一个网络设备。IPVS 模式的 Cluster IP 是可以 ping 通的。

![image-20240306224527808](https://typora-1306014148.cos.ap-nanjing.myqcloud.com/markdown/image-20240306224527808.png)

根据 Kubernetes 的网络模型，一个 Node 上的 Pod 与其他 Node 上的 Pod 应该能够直接建立双向的 TCP/IP 通信通道，所以如果直接修改 iptables 规则，则也可以实现 kube-proxy 的功能，只不过后者更加高端，因为是全自动模式的。与 userspace 模式相比，iptables 模式完全工作在内核态，不用再经过用户态的 kube-proxy 之中转，因而性能更强。

kube-proxy 针对 Service 和 Pod 创建的一些主要 iptables 规则如下：

- KUBE-CLUSTER-IP：在 masquerade-all=true 或 clusterCIDR 指定的情况下对 Service ClusterIP 地址进行伪装，以解决数据包欺骗问题；
- KUBE-EXTERNAL-IP：将数据包伪装成 Service 的外部 IP 地址；
- KUBE-LOAD-BALANCER、KUBE-LOAD-BALANCER-LOCAL： 伪装 Load Balancer 类型的 Service 流量；
- KUBE-NODE-PORT-TCP、KUBE-NODE-PORT-LOCAL-TCP、KUBE-NODE-PORT-UDP、KUBE-NODE-PORT-LOCAL-UDP：伪装 NodePort 类型的 Service 流量；

**IPVS**

iptables 模式实现起来虽然简单，性能也提升很多，但存在固有缺陷，在集群中的 Service 和 Pod 大量增加以后，每个 Node 节点上 iptables 中的规则会急速膨胀，导致网络性能显著下降，在某些极端情况下甚至会出现规则丢失的情况，并且这种故障难以重现与排查。于是 kubernetes 从 1.8 版本开始引入 IPVS（IP Virtual Server）模式，如下图所示。IPVS 在 Kubernetes 1.11 版本中升级为 GA 稳定版本。

![image-20240306230109652](https://typora-1306014148.cos.ap-nanjing.myqcloud.com/markdown/image-20240306230109652.png)

IPVS 相对 iptables 效率会更高一些，使用 IPVS 模式需要在运行 kube-proxy 的节点上安装 ipvsadm、ipset 工具包和加载 ip_vs 内核模块，当 kube-proxy 以 IPVS 代理模式启动时，kube-proxy 将验证节点上是否安装了 IPVS 模块，如果未安装，kube-proxy 将会退到 iptables 代理模式。

使用 IPVS 模式，kube-proxy 会监视 Kubernetes Service 对象和 Endpoints，调用宿主机内核 Netlink 接口以相应地创建 IPVS 规则并定期与 Kubernetes Service 对象和 Endpoints 对象同步 IPVS 规则，以确保 IPVS 状态与期望一致，访问服务时，流量将被重定向到其中一个后端 Pod，IPVS 使用哈希表作为底层数据结构并在内核空间中工作，这意味着 IPVS 可以更快地重定向流量，并且在同步代理规则时具有更好的性能，此外，IPVS 为负载均衡算法提供了更多选项，例如：rr（轮询调度）、lc（最小连接数）、dh（目标哈希）、sh（源哈希）、sed（最短期望延迟）、nq（不排队调度）等。

iptables 与 IPVS 虽然都是基于 Netfilter实现的，但因为定位不同，二者有着本质的差别：iptables 时为防火墙设计的；IPVS 专门用于高性能负载均衡，并使用更高效的数据结构（哈希表），允许几乎无限的规模扩张；

**与 iptables 相比，IPVS 拥有一下明显优势**：

- 为大型集群提供了更好的可扩展性和性能；
- 支持比 iptables 更复杂的负载均衡算法（最小负载、最少连接、加权等）；
- 支持服务器健康检查和连接重试等功能；
- 可以动态修改 ipset 的集合，即使 iptables 的规则正在使用这个集合。

由于 IPVS 无法提供包过滤、airpin-masquerade tricks（地址伪装）、SNAT 等功能，因此在某些场景（如 NodePort 的实现）下还要与 iptables 搭配使用。在 IPVS 模式下，kube-proxy 又做了重要的升级，即使用 iptables 的扩展 ipset，而不是直接调用 iptables 来生成规则。

iptables 规则链是一个线性数据结构，ipset 则引入了带索引的数据结构，因此当规则很多时，也可以高效地查找和匹配。我们可以将 ipset 简单理解为一个 IP 或一个 IP 段的集合，这个集合的内容可以是 IP 地址、IP 网段、端口等，iptables 可以直接添加规则对这个**可变的集合**进行操作，这样做的好处在与大大减少了 iptables 规则的数量，从而减少了性能损耗。假设要禁止上万个 IP 访问我们的服务器，用 iptables 的话，就需要一条一条的添加规则，会在 iptables 中生成大量的规则；但是用 ipset 的话，只需要将相关的 IP 地址（网段）加入 ipset 集合中即可，这样只需要设置少量的 iptables 规则即可实现目标。

### 4.3、runtime（容器运行时）

Kubernetes Node（kubelet）的主要功能就是启动和停止容器的组件，我们称之为 Container Runtime（容器运行时），其中最知名的就是 Docker 了，为了更具扩展性，Kubernetes 从 1.5 版本开始就加入了容器运行时插件 API，即 Container Runtime Interface，简称：CRI。

每个容器运行时都有特点，因此 Kubernetes 为了能够支持更多的容器运行时，Kubernetes 从 1.5 版本开始引入了 CRI 接口规范，通过插件接口模式，Kubernetes 无需重新编译就可以使用更多的容器运行时。CRI 包含了 Protocol Buffers、gRPC API、运行库支持以及开发中的标准规范和工具。Docker 的 CRI 实现在 Kubernets 1.6 中被更新为 Beta 版本，并在 kubelet 启动时默认启动。

**CRI的主要组件**

kubelet 使用 gRPC 框架通过 UNIX Socket 与容器运行时（或 CRI 代理）进行通信，在这个过程中 kubelet 是客户端，CRI 代理（shim）时服务端，如下图所示：

![image-20240306234951888](https://typora-1306014148.cos.ap-nanjing.myqcloud.com/markdown/image-20240306234951888.png)

Protocol Buffers API 包含了两个 gRPC 服务：ImageService 和 RuntimeService

- ImageService：提供了从仓库中拉取镜像、查看和移除镜像的功能；
- RuntimeService：负责 Pod 和容器的生命周期管理、以及与容器的交互（exec/attach/port-forward）。rkt 和 Docker 这样的容器运行时可以使用一个 Socket 同时提供两个服务，在 kubelet 中可以用 --container-runtime-endpoint 和 --image-service-endpoint 参数设置这个 Socket。
- 目前已经有多款开源 CRI 项目可用于 kubernetes：Docker、CRI-O、Containerd、frakti（基于 Hypervisor 的容器运行时），各 CRI 运行时的安装手册可参考官网的说明

## 五、其他组件介绍

### 5.1、Calico网络组件

Calico官网：https://www.projectcalico.org/

Calico 是一个符合 CNI 标准的网络插件，给每个 Pod 生成一个唯一的 IP 地址，并且把每个 Node 节点当作一个路由器，并且在 kubernetes 中可以配置网络策略。

Calico 是一个基于 BGP 的纯三层的网络方案，与 OpenStack、Kubernetes、AWS、GCE 等云平台都能够良好的集成。为虚拟机及容器提供多主机间通信，没有使用重叠网络（如 flannel）驱动，采用虚拟路由代替虚拟交换。Calico 在每个计算节点都利用 Linux Kernel 实现了一个高效的 vRouter 来负责数据转发。每个 vRouter 都通过 BGP 协议把在本节点上运行的容器的传播可达信息（路由）向整个 Calico 网络广播，并自动设置到达节点的路由转发规则。而且无缝集成像 OpenStackk 这种 Laas 云架构，能够提供可控的 VM、容器、裸机之间的 IP 通信。为甚么说是纯三层的呢？因为 Calico 保证所有容器之间的数据包都是通过 IP 路由的方式完成互联互通的。然后通过 BGP 协议将所有的路由同步到所有的机器或数据中心，从而完成整个网络的互联。不需要额外的 NAT、隧道或者 Overlay Network，没有额外的封包解包，能够节约 CPU 运算，提供网络效率。简单来说，Calico 在主机上创建了一堆的 vethpair，其中一端在主机上，另一端在容器的网络命名空间里，然后在容器和主机中分别设置几条路有，来完成整个网络的互联。

此外，Calico 基于 iptables 还提供了丰富而灵活的网络 Policy，保证通过各个节点上的 ACLs 来提供 Workload 的多租户隔离、安全组以及其他可达性限制等功能。

#### 5.1.1、Calico架构

![image-20240307235847091](https://typora-1306014148.cos.ap-nanjing.myqcloud.com/markdown/image-20240307235847091.png)

Calico 主要由 Felix、etcd、BGP client 以及 BGP Route Reflector 组成

1. **Felix**：Calico Agent，运行在每台 Node 节点上，主要负责为容器设置网络资源（IP 地址、路由规则、iptables 规则、ACLs等）信息来确保 Endpoint 的连通状态；
2. **etcd**：Calico 使用的分布式键值存储，主要负责网络元数据一致性，确保 Calico 网络状态的准确性；
3. **BGP Client（BIRD）**：主要负责把 Felix 在各 Node 上写入 Kernel 的路由信息通过 BGP 广播到当前 Calico 网络，确保 Node 之间的通信的有效性；
4. **BGP Route Reflector（BIRD）**：大规模部署时使用，摒弃所有节点互联的 mesh 模式，通过一个或多个 BGP Router Reflector 来完成大规模集群的集中式（分级）路由分发；
5. **Calico/Calico-ipam**：主要用作 Kubernetes 的 CNI 插件；
6. **calicoctl**：Calico 命令行管理工具；



### 5.2、集群 DNS 组件 coredns

官方网站：https://coredns.io/

主要负责把集群中 Service 资源的名称解析成对应的 IP 地址，coredns 的副本数量可以根据集群的规模进行适当的调整，但是在集群规模达到一定数据时，扩展 coredns 副本数量并不可以提高性能，就需要部署 DNS 缓存，具体原理就是每个 Node 节点都运行一个 DNS 组件代理到 coredns，Node 节点的 Pod 的 DNS 服务器都指定到 DNS 缓存，由 DNS 缓存回复 Pod 的 DNS 请求。

![image-20240308160834744](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20240308160834744.png)

### 5.3、metrics-server

Metrics Server 是集群范围资源使用数据的聚合器。从 Kubernetes 1.8 开始，它作为 Deployment 对象，被默认部署在由 **kube-up.sh** 脚本创建的集群中。如果使用不同的 Kubernetes 安装方法，则可以使用提供的 Deployment yaml 来部署。它在 Kubernetes 1.7+ 中得到支持。

Metrics Server 从每个节点上的 kubelet 公开的 Summary API 中采集指标信息；

通过在主 API Server 中注册的 Metrics Server Kubernetes 聚合器来采集指标信息，这是在 Kubernetes 1.7 中引入的。

```bash
# 查看 node 节点使用情况
[root@k8s-master-1 ~]# kubectl top nodes 
NAME            CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
10.202.43.114   150m         2%     667Mi           4%        
10.202.43.191   172m         2%     731Mi           4%        
10.202.43.46    339m         4%     1524Mi          10%       
10.202.43.55    351m         4%     1621Mi          11%       
10.202.43.56    214m         2%     892Mi           6%        
```

### 5.4、Kubernetes Dashboard

官方网站：https://github.com/kubernetes/dashboard

Kubernetes 官方的 web 控制端工具，图形化工具