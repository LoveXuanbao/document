## 一、什么是 RC

官方网址：https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/replicationcontroller/#how-a-replicationcontroller-works

**ReplicationController** （复制控制器，简称 RC）可确保在任何时候都有特定数量的 Pod 副本处于运行状态，达到期望值。换句话说，ReplicationController 确保一个 Pod 或一组同类的 Pod 总是可用的。

**ReplicationController 如何工作**

当存在的 Pod 数量过多十，ReplicationController 会终止多余的 Pod。当 Pod 数量太少时，ReplicationController 将会启动新的 Pod。与手动创建的 Pod 不同，由 ReplicationController 创建的 Pod 在失败、被删除或被终止时会被自动替换。例如，在中断性维护（如升级内核）之后，你的 Pod 会在节点上重新创建。因此，即使应用程序只需要一个 Pod，也应该使用 ReplicationController 创建 Pod。ReplicationController 类似于进程管理器，但时 ReplicationController 不是监控单个节点上的进程，而是监控跨多个节点的多个 Pod。

**ReplicationController 的职责**

ReplicationController 仅确保所需的 Pod 数量与其标签选择算符匹配，并且是可操作的。目前，它的计数中只排除终止的 Pod。未来，可能会考虑系统提供的就绪状态和其他信息，可能会对替换策略添加更多控制。计划发出事件，这些事件可以被外部客户端用来实现任意复杂的替换和/或缩减策略。

ReplicationController 永远被限制在这个狭隘的职责范围内。它本身既不执行就绪探测，也不执行活跃性探测。它不负责执行自动扩缩容，而是由外部自动扩缩器控制，后者负责更改其 **replicas** 字段值。我们不会向 ReplicationController 添加调度策略（例如：spreading）。它也不应该验证所控制的 Pod 是否与当前指定的模版匹配，因为这会阻碍自动调整大小和其他自动化过程。类似地，完成期限、整理依赖关系、配置扩展和其他特性也属于其他地方。我们甚至计划考虑批量创建 Pod 的机制。

ReplicationController 旨在成为可组合的构建基元。希望在它和其他补充原语的基础上构建更高级别的 API 或者工具，以便用户使用。kubectl 目前支持 "macro" 操作（运行、扩缩、滚动更新）就是这方面的概念示例。例如，可以想象类似于 Asgard 的东西管理 ReplicationController、自动定标器、服务、调度策略、金丝雀发布等。

## 二、RC 的替代方案

### 2.1、ReplicaSet

ReplicaSet 是下一代的 ReplicationController，支持新的**基于集合的标签选择算符**。它主要被 Deployment 用来作为一种编排 Pod 创建、删除及更新的机制。请注意，我们推荐使用 Deployment 而不是直接使用 ReplicaSet，除非需要自定义编排或根本不需要更新。

### 2.2、Deployment（推荐）

Deployment 是一种更高级别的 API 对象，用于更新其底层 ReplicaSet 及其 Pod。如果想要这种滚动更新功能，那么推荐使用 Deployment，因为它们是声明式的、服务端的，并且具有其他特性。

### 2.3、裸 Pod

与用户直接创建 Pod 的情况不同，ReplicationController 能够替换因某些原因被删除或被终止的 Pod，例如在节点故障或中断节点维护的情况下，例如内核升级。因此，建议使用 ReplicationController，即使应用程序只需要一个 Pod。可以将其看作类似于进程管理器，它只管理跨节点的多个 Pod，而不是单个节点上的单个进程。ReplicationController 将本地容器重启委托给节点上的某个代理（例如 kubelet）。

### 2.4、Job

对于预期会自动终止的 Pod（即批处理任务），使用 **Job** 而不是 ReplicationController。

### 2.5、DaemonSet

对于提供机器级功能（例如机器监控或机器日志记录）的 Pod，使用 **DaemonSet** 而不是 ReplicationController。这些 Pod 的生命期与机器的生命期绑定；它们需要在其他 Pod 启动之前在机器上运行，并且在机器准备重启启动或者关闭时安全地终止。

### 2.6、ReplicationController

## 三、RC 示例文件

```yaml
apiVersion: v1  #必填，api版本号
kind: ReplicationController   #必填，资源类型
metadata:
  name: nginx-rc  #必填，名称
spec:  
  replicas: 3  #必填，副本数
  selector:    #必填，选择器
    app: nginx-rc
  template:    #必填，以下为pod模板信息定义
    metadata:
      name: nginx-rc
      labels:
        app: nginx-rc
    spec:
      containers:
      - name: nginx
        image: repository.test.com:8444/kubernetes/nginx/nginx:1.7.9
        ports:
        - containerPort: 80
```

## 四、什么是 RS

官方网址：https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/replicaset/

**ReplicaSet 的工作原理**

ReplicaSet 是通过一组字段来定义的，包括一个用来识别可获得的 Pod 的集合的选择算符、一个用来标明应该维护的副本个数的数值、一个用来指定应该创建新 Pod 以满足副本个数条件时要使用的 Pod 模版等。每个 ReplicaSet 都通过根据需要创建和删除 Pod 以使得副本个数达到期望值，进而实现其存在价值。当 ReplicaSet 需要创建新的 Pod 时，会使用所提供的 Pod 模版。

ReplicaSet 的目的是维护一组在任何时候都处于运行状态的 Pod 副本的稳定集合。因此，它通常用来保证给定数量的、完全相同的 Pod 的可用性。

ReplicaSet 通过 Pod 上的 **metadata.ownerReferences** 字段连接到附属 Pod，该字段给出当前对象的属主资源。ReplicaSet 所获得的 Pod 都在其 **ownerReferences** 字段中包含了属主 ReplicaSet 的标识信息。正是通过这一连接，ReplicaSet 知道它所维护的 Pod 集合的状态，并据此计划其操作行为。

ReplicaSet 使用其选择算符来辨识要获得的 Pod 集合。如果某个 Pod 没有 **OwnerReference** 或者其 **OwnerReference** 不是一个控制器，且其匹配到某 ReplicaSet 的选择算符，则该 Pod 立即被此 ReplicaSet 获得。

**何时使用 ReplicaSet**

ReplicaSet 确保任何时间都有指定数量的 Pod 副本在运行。然而，Deployment 是一个更高级的概念，它管理 ReplicaSet，并向 Pod 提供声明式的更新以及许多其他有用的功能。因此，建议使用 Deployment 而不是直接使用 ReplicaSet，除非需要自定义更新业务流程或者根本不需要更新。

这实际上意味着，你可能永远不需要操作 ReplicaSet 对象；而是使用 Deployment，并在 spec 部分定义你的应用。

## 五、RS 的替代方案

### 5.1、Deployment（推荐）

Deployment 是一个可以拥有 ReplicaSet 并使用声明式方式在服务器端完成对 Pod 滚动更新的对象。尽管 ReplicaSet 可以独立使用，目前它们的主要用途是提供给 Deployment 作为编排  Pod 创建、删除和更新的一种机制。当使用 Deployment 时，你不必关心如何管理它所创建的 ReplicaSet，Deployment 拥有并管理其 ReplicaSet。因此建议在需要 ReplicaSet 时使用 Deployment。

### 5.2、裸 Pod

与用户直接创建 Pod 的情况不同，ReplicaSet 会替换那些由于某些原因被删除或被终止的 Pod。例如在节点故障或破坏性的节点维护（如内核升级）的情况下。因为这个原因，建议使用 ReplicaSet，即使应用程序只需要一个 Pod。想象一下，ReplicaSet 类似于进程监视器，只不过它在多个节点上监视多个 Pod，而不是在单个节点上监视单个进程。ReplicaSet 将本地容器重启的任务委托给了节点上的某个代理（例如：kubelet）去完成。

### 5.3、Job

使用 **Job** 代替 ReplicaSet，可以用于那些期望自动终止的 Pod。

### 5.4、DaemonSet

对于管理那些提供主机级别功能（如主机监控和主机日志）的容器，就要用 **DaemonSet** 而不用 ReplicaSet。这些 Pod 的寿命与主机寿命有关；这些 Pod 需要先于主机上的其他 Pod 运行，并且在机器准备重新启动/关闭时安全的终止。

### 5.5、ReplicationController

ReplicaSet 是 ReplicationController 的后继者。二者目的相同且行为类似，只是 ReplicationController 不支持基于集合的选择算符需求。因此，相比于 ReplicationController，应优先考虑 ReplicaSet。

## 六、RS 示例文件

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-rs
spec:
  replicas: 3
  selector:
    matchLabels:
      tier: nginx-rs
  template:
    metadata:
      labels:
        tier: nginx-rs
    spec:
      containers:
      - name: nginx-rs
        image: repository.test.com:8444/kubernetes/nginx/nginx:1.7.9
        ports:
        - containerPort: 80
```

## 七、RC 和 RS 说明

Replication Controller 和 ReplicaSet 的创建删除和 Pod 并无太大的区别，Replication Controller 目前几乎已经不在生产环境中使用，ReplicaSet 也很少单独使用，都是使用更高级的资源 Deployment、DaemonSet、StatefulSet 进行管理 Pod。