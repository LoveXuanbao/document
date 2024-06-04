## 一、StatefulSet

官方文档：https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/statefulset/

StatefulSet 是用来管理有状态应用的工作负载 API 对象。StatefulSet 用来管理某个 Pod 集合的部署和扩缩容，并为这些 Pod 提供持久存储和持久标识符

和 Deployment 类似，StatefulSet 管理基于相同容器规定约束的一组 Pod，但是和 Deployment 不同的是，StatefulSet 为它们的每个 Pod 维护了一个有粘性的 ID。这些 Pod 是基于相同的规定约束来创建的，但是不能相互替换：无论怎么调度，每个 Pod 都有一个永久不变的 ID。比如定义一个名字是 Redis 的 StatefulSet，指定创建三个Pod，那么创建出来的 Pod 名字就为 Redis-0、Redis-1、Redis-2。而 StatefulSet 创建的 Pod 一般使用 **Headless Service（无头服务）**进行通信，和普通的 Service 的区别在于 Headless Service 没有 ClusterIP，它使用的是 Endpoint 进行互相通信，Headless 一般的格式为：`statefulSetName-{0..N-1}.serviceName.namespace.svc.cluster.local`

如果希望使用存储卷为工作负载提供持久存储，可以使用 StatefulSet 作为解决方案的一部分。尽管 StatefulSet 中的单个 Pod 仍有可能出现故障，单持久的 Pod 标识符是的将现有的卷于替换已失败的 Pod 的新 Pod 相匹配变得更加容易。

StatefulSet 对于需要满足以下一个或多个需求的应用程序很有价值：

- 稳定的、唯一的网络标识符；
- 稳定的、持久的存储；
- 有序的、优雅的部署和扩缩容；
- 有序的、自动的滚动更新

"稳定的"意味着 Pod 调度或重调度的整个过程是持久性的。如果应用程序不需要任何稳定的标识符或有序的部署、删除或扩缩容，则应该使用由一组无状态的副本控制器提供的工作负载来部署应用程序，比如 Deployment 或者 ReplicaSet 可能更适用于无状态应用部署需要。

**StatefulSet 限制**

- 给定 Pod 的存储必须由 **PersistentVolume Provisioner** 基于所有请求的 **storage class** 来制备，或者由管理员预先制备；
- 删除或者扩缩 StatefulSet 并不会删除它关联的存储卷。这样做是为了保证数据安全，它通常比自动清除 StatefulSet 所有相关的资源更有价值；
- StatefulSet 当前需要**无头服务（没有 clusterIP 的 Service 资源）**来负责 Pod 的网络标识。你需要负责创建此服务；
- 当删除一个 StatefulSet 时，该 StatefulSet 不提供任何终止 Pod 的保证。为了实现 StatefulSet 中的 Pod 可以有序且体面的终止，可以在删除之前将 StatefulSet 缩容到 0；
- 在默认 Pod 管理策略（OrderedReady）时使用滚动更新，可能进入需要人工干预才能修复的损坏状态。

## 二、SetatfulSet 资源文件

`vim nginx-sts.yaml`

```yaml
# 无头服务创建
apiVersion: v1
kind: Service
metadata:
  name: nginx-sts
  labels:
    app: nginx-sts
spec:
  ports:
    - port: 80
      name: nginx-sts
  clusterIP: None   # 无法服务必须设置为 None
  selector:
    name: nginx-sts

---
# StatefulSet 资源创建
apiVersion: apps/v1  # api版本
kind: StatefulSet  # 资源类型
metadata:  # 元数据信息
  name: nginx-sts
  namespace: default   # 命名空间
spec: # 定义控制器信息
  podManagementPolicy: OrderedReady # 必须，Pod 扩缩策略，默认为 PrderedReady
  replicas: 3  # 副本数
  revisionHistoryLimit: 10  # 必须，历史版本保存的最大数量，默认为 10
  selector:  # 选择器配置，选择 Pod
    matchLabels:  # 标签选择器
      app: nginx-sts
  serviceName: nginx-sts # 管理这个 StatefulSet 的 service 名称
  template: # pod 模板配置
    metadata:
      creationTimestamp: null
      labels:
        app: nginx-sts
    spec:
      containers:
      - image: repository.test.com:8444/kubernetes/nginx/nginx:1.24.0
        imagePullPolicy: IfNotPresent
        name: nginx-sts
        ports:
        - containerPort: 80
          name: nginx-sts
          protocol: TCP
  updateStrategy:  # 更新策略
    rollingUpdate:  # RollingUpdate更新策略才需要
      partition: 0 #表示在滚动更新时保留的pod编号，为0即表示保留小于0的，即不保留
    type: RollingUpdate #类型默认为RollingUpdate滚动更新，其他参数Ondelete在更新pod后只有删除pod才会触发更新
```

### 2.1、创建 StatefulSet

```bash
[root@k8s-master-1 study]# kubectl create -f nginx-sts.yaml 
service/nginx-sts created
statefulset.apps/nginx-sts created
```

- 查看 StatefulSet

```bash
[root@k8s-master-1 study]# kubectl get statefulset
NAME        READY   AGE
nginx-sts   3/3     45s
```

- 查看 service

```bash
[root@k8s-master-1 study]# kubectl get service 
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   172.10.0.1   <none>        443/TCP   4d6h
nginx-sts    ClusterIP   None         <none>        80/TCP    90s
```

- 查看 Pod

```bash
[root@k8s-master-1 study]# kubectl get pod 
NAME          READY   STATUS    RESTARTS   AGE
nginx-sts-0   1/1     Running   0          119s
nginx-sts-1   1/1     Running   0          117s
nginx-sts-2   1/1     Running   0          116s
```

## 三、StatefulSet 创建 Pod 流程

StatefulSet 管理 Pod 的部署和扩展规则如下：

1. 对于具有 N 个副本的 StatefulSet，将按照顺序从 0 到 N-1 开始创建pod；
2. 当删除 Pod 时，将按照 N-1 到 0 的反顺序执行删除；
3. 在缩容 Pod 之前，必须保证当前的 Pod 是 Running 状态或者 Ready 状态；
4. 在终止 Pod 之前，它所有的继任者必须是完全关闭状态

StatefulSet 的 **pod.spec.TerminationGracePeriodSeconds** （终止 Pod 的等待时间）不应该指定为 0。设置为 0 对 StatefulSet 的 Pod 是极其不安全的做法，优雅地删除 StatefulSet 的 Pod 是非常有必要的，而且是安全的，因为它可以确保在 kubelet 从 APIServer 删除之前，让 Pod 正常关闭

当创建上面的 nginx-sts 实例的时候，Pod 将按照 `nginx-sts-0、nginx-sts-1、nginx-sts-2 的顺序创建 3 个 Pod`，在 nginx-sts-0 处于 Running 或者 Ready 之前，nginx-sts-1 不会被部署；相同的 nginx-sts-2 在 nginx-sts-1 未处于 Runing 和 Ready 之前也不会被部署。如果 nginx-sts-1 处于 Running 和 Ready 状态时，nginx-sts-0 变成 Failed（失败）状态，那么 nginx-sts-2 将不会部署，直到 nginx-sts-0 恢复为 Running 和 Ready 状态。

如果用户将 StatefulSet 的 replicas 设置为 1，那么 nginx-sts-2 将首先被终止，在完全关闭并删除 nginx-sts-2 之前，不会删除 nginx-sts-1；如果 nginx-sts-1 终止并完全关闭后， nginx-sts-0 突然失败，那么在 nginx-sts-0 未恢复成 Running 或 Ready 时，nginx-sts-1 不会被删除

## 四、StatefulSet 扩容和缩容

StatefulSet 的扩容和缩容与 Deployment 类似，可以通过更新 replicas 字段进行扩容和缩容，也可以使用 `kubectl scale`、`kubectl edit` 和 `kubectl patch` 来扩容和缩容 StatefulSet

- 扩容

```bash
[root@k8s-master-1 study]# kubectl scale sts nginx-sts --replicas=5
statefulset.apps/nginx-sts scaled
```

查看扩容后的 Pod 状态

```bash
[root@k8s-master-1 study]# kubectl get pod -l app=nginx-sts
NAME          READY   STATUS    RESTARTS   AGE
nginx-sts-0   1/1     Running   0          12h
nginx-sts-1   1/1     Running   0          12h
nginx-sts-2   1/1     Running   0          12h
nginx-sts-3   1/1     Running   0          35s
nginx-sts-4   1/1     Running   0          33s
```

- 缩容

```bash
[root@k8s-master-1 study]# kubectl patch sts nginx-sts -p '{"spec":{"replicas": 3}}'
statefulset.apps/nginx-sts patched
```

查看缩容后的 Pod 状态，可以发现 `nginx-sts-4` 已经没有了，`nginx-sts-3` 处于正在删除中

```bash
[root@k8s-master-1 study]# kubectl get pod -l app=nginx-sts
NAME          READY   STATUS        RESTARTS   AGE
nginx-sts-0   1/1     Running       0          12h
nginx-sts-1   1/1     Running       0          12h
nginx-sts-2   1/1     Running       0          12h
nginx-sts-3   0/1     Terminating   0          117s
```

## 五、StatefulSet 更新策略

**OnDelete策略**：OnDelete 更新策略实现了传统（1.7版本之前）的行为，它也是默认的更新策略。当我们选择这个更新策略并修改 StatefulSet 的 `.spec.template` 字段时，StatefulSet 控制器不会自动更新 Pod，必须手动删除 Pod 才能使控制器创建新的 Pod。

**RollingUpdate策略**：RollingUpdate更新策略会自动更新一个 StatefulSet 中所有的 Pod，采用于序号索引相反的顺序进行滚动更新。默认更新策略。

**滚动更新**：当 StatefulSet 的 `.spec.updataStrategy.type` 被设置为 RollingUpdate 时，StatefulSet 控制器会删除和重建 StatefulSet 中的每个 Pod。它将按照与 Pod 终止相同的顺序（从最大序号到最小序号）进行，每次更新一个 Pod；Kubernetes 控制平面会等到被更新的 Pod 进入 Running 和 Ready 状态，然后再更新其前身。如果设置了 `.spec.minReadySeconds`（最短就绪秒数），控制平面在 Pod 就绪后会额外等待一定的时间再执行下一步。

**分区滚动更新**：通过声明 `.spec.updateStrategy.rollingUpdate.partition` 的方式，RollingUPdate 更新策略可以实现分区更新。如果声明了一个分区，当 StatefulSet 的 `.spec.template` 被更新时，所有序号索引大于等于该分区序号索引的 Pod 都会被更新。所有序号索引小雨该分区序号的 Pod 都不会被更新，并且，即使它们被删除也会依据之前的版本进行重建。如果 StatefulSet 的 `.spec.updateStrategy.rollingUpdate.partition` 大于它的 `.spec.replicas`，则对它的 `.spec.template` 的更新将不会传递到它的 Pod。在大多数情况下，不需要使用分区。但如果希望进行阶段更新或执行分阶段上线，则分区滚动更新非常有用。 

**最大不可用 Pod**：在 1.24 版本引用，可以通过指定 `.spec.updateStrategy.rollingUpdate.maxUnavailabel` 字段控制更新期间不可用的 Pod 的最大数量。该值可以是绝对值（例如：5）或者是期望 Pod 个数的百分比（例如：10%）。绝对值是根据百分比值四舍五入计算的。该字段不能为 0，默认为 1。该字段适用于 **0 到 Replicas -1** 范围内的所有 Pod。如果在 **0 到 Replicas -1** 范围内存在不可用 Pod，这类 Pod 将被计入 `maxUnavailable` 值。【maxUnavailabel 字段处于 Alpha 阶段，仅当 API 服务器启用了 MaxUnavailabelStatefulSet 特性门控时才起作用】

### 5.1、查看默认更新策略

```bash
[root@k8s-master-1 study]# kubectl get sts nginx-sts -o yaml | grep  updateStrategy -A 3
  updateStrategy:
    rollingUpdate:
      partition: 0
    type: RollingUpdate  # 默认更新策略为 RollingUpdate
```

### 5.2、RollingUpdate 更新演示

- 对于多个副本的 StatefulSet，在更新 Pod 时会依次 N-1 到 0 的逐个更新 Pod；
- 如果在更细过程中前面运行的 Pod 出现问题会暂停 Pod 更新，等待之前的 Pod 启动完成之后，重新在接着进行更新。

```bash
[root@k8s-master-1 test]# kubectl set env sts nginx-sts test=123
statefulset.apps/nginx-sts env updated
[root@k8s-master-1 test]# kubectl get pod -w
NAME          READY   STATUS        RESTARTS   AGE
nginx-sts-0   1/1     Running       0          7m59s
nginx-sts-1   1/1     Running       0          7m57s
nginx-sts-2   0/1     Terminating   0          7m57s   # 先停止 nginx-sts-2
nginx-sts-2   0/1     Pending       0          0s      # 然后创建 nginx-sts-2
nginx-sts-2   0/1     ContainerCreating   0          0s
nginx-sts-2   1/1     Running             0          2s
nginx-sts-1   0/1     Terminating         0          8m8s   # 在停止 nginx-sts-1
nginx-sts-1   0/1     Pending             0          0s     # 然后创建 nginx-sts-1
nginx-sts-1   0/1     ContainerCreating   0          0s
nginx-sts-1   1/1     Running             0          1s
nginx-sts-0   0/1     Terminating         0          8m20s  # 在停止 nginx-sts-0
nginx-sts-0   0/1     Pending             0          0s     # 然后创建 nginx-sts-0
nginx-sts-0   0/1     ContainerCreating   0          0s
nginx-sts-0   1/1     Running             0          1s
```

### 5.3、OnDelete 更新演示

- 修改更新策略为 **OnDelete**

```bash
vim nginx-sts.yaml

...
  updateStrategy:  # 更新策略
    type: OnDelete
...

[root@k8s-master-1 ~]# kubectl replace -f nginx-sts.yaml   # 使用修改后的 yaml 文件来替换资源
service/nginx-sts replaced
statefulset.apps/nginx-sts replaced

# 查看更新策略
[root@k8s-master-1 ~]# kubectl get sts nginx-sts -oyaml  | grep updateStrategy -A 2
  updateStrategy:
    type: OnDelete
```

- 更新演示

```bash
[root@k8s-master-1 ~]# kubectl set image sts nginx-sts nginx-sts=repository.test.com:8444/kubernetes/nginx/nginx:1.7.9  # 修改 image 镜像
statefulset.apps/nginx-sts env updated
[root@k8s-master-1 ~]# kubectl get pod -l app=nginx-sts  # 查看 Pod，可以看到，没有触发更新
NAME          READY   STATUS    RESTARTS   AGE
nginx-sts-0   1/1     Running   0          19m
nginx-sts-1   1/1     Running   0          20m
nginx-sts-2   1/1     Running   0          20m

[root@k8s-master-1 ~]# kubectl delete pod nginx-sts-1  # 手动删除 nginx-sts-1
pod "nginx-sts-1" deleted
[root@k8s-master-1 ~]# kubectl get pod -l app=nginx-sts
NAME          READY   STATUS    RESTARTS   AGE
nginx-sts-0   1/1     Running   0          20m
nginx-sts-1   1/1     Running   0          3s   # 可以看到只有 nginx-sts-1 才更新
nginx-sts-2   1/1     Running   0          21m
[root@k8s-master-1 ~]# kubectl get pod -l app=nginx-sts -o yaml | grep image:
      image: repository.test.com:8444/kubernetes/nginx/nginx:1.24.0
      image: repository.test.com:8444/kubernetes/nginx/nginx:1.24.0
    - image: repository.test.com:8444/kubernetes/nginx/nginx:1.7.9  # 只有重启过的 pod 才更新了镜像
      image: repository.test.com:8444/kubernetes/nginx/nginx:1.7.9
      image: repository.test.com:8444/kubernetes/nginx/nginx:1.24.0
      image: repository.test.com:8444/kubernetes/nginx/nginx:1.24.0
```

### 5.4、分区滚动更新

```bash
# 定义一个分区 **"partition": 3** ，可以使用 patch 或者 edit 直接对 StatefulSet 进行设置：
[root@k8s-master-1 ~]# kubectl patch sts nginx-sts -p '{"spec":{"updateStrategy":{"type": "RollingUpdate", "rollingUpdate":{"partition":3}}}}'
statefulset.apps/nginx-sts patched

# 修改镜像版本
[root@k8s-master-1 test]# kubectl set image sts nginx-sts nginx-sts=repository.test.com:8444/kubernetes/nginx/nginx:1.7.9
statefulset.apps/nginx-sts image updated

# 因为运行的 Pod 的序号都小于分区 3， 所以 Pod 不会自动更新
[root@k8s-master-1 test]# kubectl get pod -l app=nginx-sts
NAME          READY   STATUS    RESTARTS   AGE
nginx-sts-0   1/1     Running   0          74s
nginx-sts-1   1/1     Running   0          72s
nginx-sts-2   1/1     Running   0          70s

# 手动删除 nginx-sts-2 的 Pod
[root@k8s-master-1 test]# kubectl delete pod nginx-sts-2
pod "nginx-sts-2" deleted
[root@k8s-master-1 test]# kubectl get pod -l app=nginx-sts
NAME          READY   STATUS    RESTARTS   AGE
nginx-sts-0   1/1     Running   0          107s
nginx-sts-1   1/1     Running   0          105s
nginx-sts-2   1/1     Running   0          7s

# 查看镜像，为删除的 nginx-sts-2 小于分区 3，使用的还是原来的镜像，
[root@k8s-master-1 ~]# kubectl get pod nginx-sts-2 -o yaml | grep image:  -A 2
  - image: repository.test.com:8444/kubernetes/nginx/nginx:1.24.0
    imagePullPolicy: IfNotPresent
    name: nginx-sts
    
# 定义分区为 1
[root@k8s-master-1 ~]# kubectl patch sts nginx-sts -p '{"spec":{"updateStrategy":{"type": "RollingUpdate", "rollingUpdate":{"partition":1}}}}'
statefulset.apps/nginx-sts patched

# 查看更新过程
[root@k8s-master-1 ~]# kubectl get pod -l app=nginx-sts -w
NAME          READY   STATUS        RESTARTS   AGE
nginx-sts-0   1/1     Running       0          8m35s
nginx-sts-2   0/1     Terminating   0          8m33s  # 开始滚动更新
nginx-sts-2   0/1     Pending       0          0s
nginx-sts-2   0/1     ContainerCreating   0          0s
nginx-sts-2   1/1     Running       0          5s
nginx-sts-1   0/1     Terminating   0          8m38s
nginx-sts-1   0/1     Pending       0          0s
nginx-sts-1   0/1     ContainerCreating   0          0s
nginx-sts-1   1/1     Running             0          1s

# nginx-sts-2 和 nginx-sts-1 已更新完毕，nginx-sts-0 还保持原有的
[root@k8s-master-1 test]# kubectl get pod -l app=nginx-sts -o yaml | grep image: -A 2
    - image: repository.test.com:8444/kubernetes/nginx/nginx:1.24.0
      imagePullPolicy: IfNotPresent
      name: nginx-sts
    - image: repository.test.com:8444/kubernetes/nginx/nginx:1.7.9
      imagePullPolicy: IfNotPresent
      name: nginx-sts
    - image: repository.test.com:8444/kubernetes/nginx/nginx:1.7.9
      imagePullPolicy: IfNotPresent
      name: nginx-sts
```

## 六、StatefulSet 删除

删除 StatefulSet 有两种方式，**级联删除**和**非级联删除**。使用非级联方式删除 StatefulSet 时，StatefulSet 的 Pod 不会被删除；使用级联删除时，StatefulSet 和它的 Pod 都会被删除。

### 6.1、非级联删除

使用 **kubectl delete sts xxx** 删除 StatefulSet 时，只需要提供 **--cascade=false** 参数，就会采用非级联删除，此时删除 StatefulSet，不会删除它的 Pod

```bash
[root@k8s-master-1 ~]# kubectl get pod 
NAME          READY   STATUS    RESTARTS   AGE
nginx-sts-0   1/1     Running   0          177m
nginx-sts-1   1/1     Running   0          168m
nginx-sts-2   1/1     Running   0          168m
[root@k8s-master-1 ~]# kubectl delete sts nginx-sts --cascade=false  # 采用非级联删除
statefulset.apps "nginx-sts" deleted
[root@k8s-master-1 ~]# kubectl get sts   # 查看 statefulset，已经被删除
No resources found in default namespace.
[root@k8s-master-1 ~]# kubectl get pod   # 该 statefulset 管理的 Pod 并未被删除
NAME          READY   STATUS    RESTARTS   AGE
nginx-sts-0   1/1     Running   0          178m
nginx-sts-1   1/1     Running   0          169m
nginx-sts-2   1/1     Running   0          169m
```

由于此时删除了 StatefulSet，它管理的 Pod 变成了 “孤儿 Pod”，因此单独删除 Pod 时，该 Pod 不会被重建

```bash
[root@k8s-master-1 ~]# kubectl delete pod nginx-sts-0 nginx-sts-1 nginx-sts-2 
pod "nginx-sts-0" deleted
pod "nginx-sts-1" deleted
pod "nginx-sts-2" deleted
[root@k8s-master-1 ~]# kubectl get pod
No resources found in default namespace.
```

### 6.2、级联删除

省略 **--cascade=false** 参数即为级联删除

```bash
[root@k8s-master-1 test]# kubectl get pod 
NAME          READY   STATUS    RESTARTS   AGE
nginx-sts-0   1/1     Running   0          18m
nginx-sts-1   1/1     Running   0          18m
nginx-sts-2   1/1     Running   0          18m
[root@k8s-master-1 test]# kubectl get sts
NAME        READY   AGE
nginx-sts   3/3     18m
[root@k8s-master-1 test]# kubectl delete sts nginx-sts  # 级联删除
statefulset.apps "nginx-sts" deleted
[root@k8s-master-1 test]# kubectl get pod   # 可以看到 Pod 已经正在终止
NAME          READY   STATUS        RESTARTS   AGE
nginx-sts-0   0/1     Terminating   0          19m
nginx-sts-1   0/1     Terminating   0          19m
nginx-sts-2   0/1     Terminating   0          19m
[root@k8s-master-1 test]# kubectl get pod 
No resources found in default namespace.
```

也可以使用 **-f** 指定创建的 StatefulSet 和 Service 的文件，之间删除 StatefulSet 和 Service（此文件将 StatefulSet 和 Service 写在了一起）

```bash
[root@k8s-master-1 test]# kubectl delete -f nginx-sts.yaml 
service "nginx-sts" deleted
# 因为 StatefulSet 已经被删除，所以会提示该 StatefulSet 不存在
Error from server (NotFound): error when deleting "nginx-sts.yaml": statefulsets.apps "nginx-sts" not found
```





