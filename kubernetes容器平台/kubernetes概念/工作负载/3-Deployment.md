## 一、什么是 Deployment

官方文档：https://kubernetes.io/zh/docs/concepts/workloads/controllers/deployment/

对于 Kubernetes 来说 Pod 是资源调度的最小单元，Kuberentes 主要的功能就是管理多个 Pod， Pod 中可以包含一个或者多个容器。而 Deployment 就是为了管理 Pod 的一个资源管理对象；

Deployment 使得 Pod 和 ReplicaSet 能够进行声明式更新；

Deployment 用于部署无状态的服务，这个是最常用的控制器。一般用于管理维护企业内部无状态的微服务，比如**configserver、zuul、springboot 等**。它可以管理多个副本的 Pod，实现无缝迁移、自动扩容缩容、自动灾难恢复、一键回滚等功能。

## 二、Deployment 的工作方式

Deploymen 通过管理 ReplicaSet 来实现应用程序的部署和更新，提供了更高级别的抽象，简化了操作和管理的复杂性。ReplicaSet 则负责确保指定数量的 Pod 副本在集群中运行，并处理 Pod 的创建、更新和删除。

1. 用户创建一个 Deployment 对象，指定应用程序的镜像、副本数量、容器端口等配置信息；
2. Deployment 控制器会创建一个或多个 ReplicaSet，用于管理 Pod 的副本数量和健康状态；
3. 当用户对 Deployment 对象进行更新时，Deployment 控制器会检测到变化，并创建新的 ReplicaSet 来管理新版本的 Pod 副本；
4. Deployment 控制器会逐步调整 Pod 的副本数量，同时在新的 ReplicaSet 中创建新的 Pod 副本，并逐步关闭旧的 ReplicaSet 中的 Pod 副本，以实现应用程序的平滑升级。
5. 如果用户需要回滚到先前的版本，Deployment 控制器会自动切换回旧的 ReplicaSet，保证应用程序的稳定性。

## 三、使用命令方式生成一个deployment

```bash
kubectl create deploy nginx --image=repository.test.com:8444/kubernetes/nginx/nginx:1.7.9 --replicas=3 -oyaml --dry-run=client > nginx-deploy.yaml
```

- 查看生成的deployment，也可以自己手动编写

```bash
[root@k8s-master-1 study]# cat nginx-deploy.yaml
apiVersion: apps/v1  # api版本
kind: Deployment # 控制器类型
metadata:  # 元数据信息
  creationTimestamp: null
  labels:
    app: nginx
  name: nginx  # Deployment名称
spec:
  replicas: 3  # 预期 Pod 的数量
  selector:  # 必需，提供 Pod 所用的标签选择符，通过此字段选择现有的 ReplicaSet 的 Pod 集合
    matchLabels:
      app: nginx
  strategy: {}  # 将现有 Pod 替换为新 Pod 时所用的部署策略
  template:  # 描述将要创建的 Pod template.spec.restartPolicy 唯一被允许的值是 always
    metadata:
      creationTimestamp: null
      labels:
        app: nginx  # 用来标记 Pod
    spec:  # 定义Pod的信息
      containers:
      - image: repository.test.com:8444/kubernetes/nginx/nginx:1.7.9
        name: nginx
        resources: {}
status: {}   # 最近观测到的 Deployment 状态
```

**使用 kubectl create -f nginx-deploy.yaml 创建 Deployment**

```bash
[root@k8s-master-1 study]# kubectl create -f nginx-deploy.yaml 
deployment.apps/nginx created
```

**使用 kubectl get 或者 kubectl describe 查看 Deployment 的状态**

```bash
[root@k8s-master-1 study]# kubectl get deployment
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   3/3     3            3           119s
```

在检查集群中 Deployment 时，所显示的字段有：

- **NAME**：集群中 Deployment 的名称；
- **READY**：Pod 就绪个数和总副本数；
- **UP-TO-DATE**：显示以达到期望状态的被更新的副本数；
- **AVAILABLE**：显示用户可以使用的Pod 副本数；
- **AGE**：显示 Deployment 服务运行的时间;

**使用 rollout 命令查看整个 Deployment 创建状态**

```bash
[root@k8s-master-1 study]# kubectl rollout status deploy nginx
deployment "nginx" successfully rolled out
```

当 rollout 结束时，再次查看此 Deployment，可以看到 AVAILABLE 的数量和 yaml文件中定义的 replicas 相同

**查看此 Deployment 当前对应的 ReplicaSet**

```bash
[root@k8s-master-1 study]# kubectl get rs -l app=nginx
NAME               DESIRED   CURRENT   READY   AGE
nginx-64f5bcd79f   3         3         3       6m26s
```

在检查 ReplicaSet 时，所显示的字段有：

- **NAME**：列出命名空间中 ReplicaSet 的名称；

- **DESIRED**：显示应用的期望副本个数，即在创建 Deployment 时所定义的值，此为期望状态；
- **CURRENT**：显示当前正在运行的副本个数；
- **READY**：显示应用中有多少副本可以为用户提供服务；
- **AGE**：显示应用已经运行的时间长度；

> ReplicaSet 的名称格式始终为 [Deployment 名称]-[哈希]。该名称将成为所创建的 Pod 的命名基础。其中的 **哈希** 字符串与 ReplicaSet 上的 **pod-template-hash** 标签一致。

当 Deployment 有过更新，对应的 RS 可能不止一个，可以通过 **-o yaml** 获取当前对应的 RS 是哪个，其余的 RS 为保留的历史版本，用于回滚等操作。

查看此 Deployment 创建的 Pod，可以看到 Pod 的 Hash 值 64f5bcd79f 和上述 Deployment 对应的 ReplicaSet 的 Hash 值一致

```bash
[root@k8s-master-1 study]# kubectl get pod --show-labels
NAME                     READY   STATUS    RESTARTS   AGE   LABELS
nginx-64f5bcd79f-62tb4   1/1     Running   0          11m   app=nginx,pod-template-hash=64f5bcd79f
nginx-64f5bcd79f-kfr9c   1/1     Running   0          11m   app=nginx,pod-template-hash=64f5bcd79f
nginx-64f5bcd79f-tv75d   1/1     Running   0          11m   app=nginx,pod-template-hash=64f5bcd79f/
```

## 四、更新 Deployment

> 更新 Deployment 只有在当前 Pod 模版（.spec.template）更改时，才会触发 Deployment 更新，例如更改内存、CPU配置、配置文件挂载或者容器镜像。其他更新（如对 Deployment 执行扩缩容的操作）不会触发上线动作。

更新 nginx 的 Deployment 的镜像使用 **repository.test.com:8444/kubernetes/nginx/nginx:1.24.0**，并使用 **--record** 记录当前更改的参数，后期回滚时可以查看到对应的信息

```bash
[root@k8s-master-1 ~]# kubectl set image deploy nginx nginx=repository.test.com:8444/kubernetes/nginx/nginx:1.24.0 --record

# 输出类似于：
deployment.apps/nginx image updated
```

也可以使用 **edit 直接编辑 deployment**， 效果是一样的

```bash
[root@k8s-master-1 study]# kubectl edit deploy nginx

# 输出类似于：
deployment.apps/nginx edited
```

同样可以使用 **kubectl rollout status** 查看更新过程

```bash
[root@k8s-master-1 ~]# kubectl rollout status deploy nginx
Waiting for deployment "nginx" rollout to finish: 1 out of 3 new replicas have been updated...
Waiting for deployment "nginx" rollout to finish: 1 out of 3 new replicas have been updated...
Waiting for deployment "nginx" rollout to finish: 1 out of 3 new replicas have been updated...
Waiting for deployment "nginx" rollout to finish: 2 out of 3 new replicas have been updated...
Waiting for deployment "nginx" rollout to finish: 2 out of 3 new replicas have been updated...
Waiting for deployment "nginx" rollout to finish: 2 out of 3 new replicas have been updated...
Waiting for deployment "nginx" rollout to finish: 1 old replicas are pending termination...
Waiting for deployment "nginx" rollout to finish: 1 old replicas are pending termination...
deployment "nginx" successfully rolled out
```

> rollout 更新过程为 新旧交替更新，首先创建一个新 Pod，当 Pod 状态为 Running 时，删除一个旧的 Pod，同时在创建一个新的 Pod。当触发一个更新后，会有新的 ReplicaSet 产生，旧的 ReplicaSet 会被保存，查看此时的 ReplicaSet，可以从 AGE 或 READY 看出来新的 ReplicaSet

```bash
[root@k8s-master-1 study]# kubectl get rs
NAME               DESIRED   CURRENT   READY   AGE
nginx-5c97db8cdc   0         0         0       9m34s
nginx-64f5bcd79f   3         3         3       3m56s
[root@k8s-master-1 study]# 
```

Deployment 可确保在更新时仅关闭一定数量的 Pod。默认情况下，它确保至少所需 Pod 的 75% 处于运行状态（最大不可用比例为 25%）；

Deployment 还确保仅所创建 Pod 的数量只可能比期望 Pod 数高一点点。默认情况下，它可确保启动的 Pod 个数比期望个数最多多出 125%（最大峰值 25%）；

在更新时，Deployment 首先创建了一个新的 Pod，然后删除旧的 Pod，并创建了新的 Pod。它不会杀死旧 Pod，直到有足够数量的新 Pod 已经出现。在足够数量的旧 Pod 被杀死钱并没有创建新 Pod。它确保至少 3 个 Pod 可用，同时最多总共 4 个 Pod 可用。当 Deployment 设置为 4 个副本时，Pod 的个数会介于 3 和 5 之间。

**通过 describe 查看 Deployment 的详细信息**

```bash
[root@k8s-master-1 study]# kubectl describe deploy nginx
...
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  84s   deployment-controller  Scaled up replica set nginx-64f5bcd79f to 3
  Normal  ScalingReplicaSet  61s   deployment-controller  Scaled up replica set nginx-5c97db8cdc to 1
  Normal  ScalingReplicaSet  60s   deployment-controller  Scaled down replica set nginx-64f5bcd79f to 2
  Normal  ScalingReplicaSet  60s   deployment-controller  Scaled up replica set nginx-5c97db8cdc to 2
  Normal  ScalingReplicaSet  48s   deployment-controller  Scaled down replica set nginx-64f5bcd79f to 1
  Normal  ScalingReplicaSet  48s   deployment-controller  Scaled up replica set nginx-5c97db8cdc to 3
  Normal  ScalingReplicaSet  46s   deployment-controller  Scaled down replica set nginx-64f5bcd79f to 0
```

> 在 describe 查看到的事件信息中，可以看出，第一次创建时，创建了一个名为 `nginx-64f5bcd79f` 的 ReplicaSet ，并直接将其扩展为 3 个副本，更新部署时，它创建了一个新的 ReplicaSet，命名为 `nginx-5c97db8cdc` ，并将其副本数扩展为 1，然后将旧的 ReplicaSet 缩小为 2，这样只要可以有 2 个 Pod 可用，最多创建了 4 个 Pod；以此类推，使用相同的滚动更新策略向上和向下扩展新旧 ReplicaSet，最终新的 ReplicaSet 达到我们期望的副本数量时，旧的 ReplicaSet 缩小为 0。 

**翻转（多 Deployment 动态更新）**

Deployment 控制器每次注意到新的 Deployment 时，都会创建一个 ReplicaSet 以启动所需的 Pod。如果更新了 Deployment，则控制标签匹配 **.spec.selector** 但模板不匹配 **.spec.template** 的 Pod 的现有 ReplicaSet 被缩容。最终，新的 ReplicaSet 缩放为 **.spce.replicas** 个副本，所有旧 ReplicaSet 缩放为 0 个副本。

当 Deployment 正在上线时被更新，Deployment 会针对更新创建一个新的 ReplicaSet 并开始对其扩容，之前正在被扩容的 ReplicaSet 会被翻转，添加到旧 ReplicaSet 列表并开始缩容。

例如：假定在创建一个 Deployment 以生成 **nginx:1.14.2** 的 5 个副本，但接下来更新 Deployment 以创建 5 个 **nginx:1.16.1** 的副本，而此时只有 3 个 **nginx:1.14.2** 副本已创建。在这种情况下，Deployment 会立即开始杀死 3 个 **nginx:1.14.2** 的 Pod，并开始创建 **nginx:1.16.1** 的 Pod。它不会等待 **nginx:1.14.2** 的 5 个副本都创建完成后才开始执行变更动作。

**更改标签选择算符**

通常不鼓励更新标签选择算符。建议提前规划选择算符。在任何情况下，如果需要更新标签选择算符，请格外小心，并确保自己了解这背后可能发生的所有事情。**在 API 版本 <apps/v1> 中，Deployment 标签选择算符在创建后是不可变的**。

- 添加选择算符时要求使用新标签更新 Deployment 规约中的 Pod 模板标签，否则将返回验证错误。此更改时非重叠的，也就是说新的选择算符不会选择使用旧选择算符所创建的 ReplicaSet 和 Pod，这会导致创建新的 ReplicaSet 时所有旧 ReplicaSet 都会被孤立；
- 选择算符的更新如果更改了某个算符的键名，这会导致于添加算符时相同的行为。
- 删除选择算符的操作会删除从 Deployment 选择算符中删除现有算符。此操作不需要更改 Pod 模板标签。现有 ReplicaSet 不会被孤立，也不会因此创建新的 ReplicaSet，但请注意已删除的标签仍然存在于现有的 Pod 和 ReplicaSet 中。

## 五、回滚 Deployment

当更新了版本不稳定或者配置不合理时，可以对其进行回滚操作（比如，镜像打的有问题，需要还原之前的版本使用的镜像）。默认情况下，Deployment 的所有上线记录都保留在系统中，以便可以随时回滚（可以通过修改修订历史记录限制来更改这一约束）。

> 说明：Deployment 被触发上线时，系统就会创建 Deployment 的新的修订版本。这意味着仅当 Deployment 的 Pod 模板（**.spce.template**）发生更改时，才会创建新修订版本；例如：模板的标签或容器镜像发生变化。其他更新，如 Deployment 的扩缩容操作不会创建 Deployment 修订版本。这是为了方便同时执行手动缩放或自动缩放。换言之，当你回滚到较早的修订版本时，只有 Deployment 的 Pod 模板部分会被回滚。

**查看 Deployment 的历史版本**

- **REVISION**：历史版本列表
- **CHANGE-CAUSE**：上线的历史记录；内容是从 Deployment 的 **kubernetes.io/change-cause** 注解复制过来的。复制动作发生在修订版本创建时。

```bash
[root@k8s-master-1 ~]# kubectl rollout history deploy nginx
deployment.apps/nginx 
REVISION  CHANGE-CAUSE
1         <none>
2         kubectl set image deploy nginx nginx=repository.test.com:8444/kubernetes/nginx/nginx:1.24.0 --record=true
```

查看 Deployment 某次更新的详细信息，使用 `--revision` 指定某次更新版本号

```bash
[root@k8s-master-1 ~]# kubectl rollout history deploy nginx --revision=2
deployment.apps/nginx with revision #2
Pod Template:
  Labels:       app=nginx
        pod-template-hash=5c97db8cdc
  Annotations:  kubernetes.io/change-cause: kubectl set image deploy nginx nginx=repository.test.com:8444/kubernetes/nginx/nginx:1.24.0 --record=true
  Containers:
   nginx:
    Image:      repository.test.com:8444/kubernetes/nginx/nginx:1.24.0
    Port:       <none>
    Host Port:  <none>
    Environment:        <none>
    Mounts:     <none>
  Volumes:      <none>
```

回滚到上个一版本，使用 **kubectl rollout undo**

```bash
[root@k8s-master-1 ~]# kubectl rollout undo deploy nginx
deployment.apps/nginx rolled back
```

再次查看更新历史，发现 REVISION 回到了上一个版本

```bash
[root@k8s-master-1 ~]# kubectl rollout history deploy nginx
deployment.apps/nginx 
REVISION  CHANGE-CAUSE
2         kubectl set image deploy nginx nginx=repository.test.com:8444/kubernetes/nginx/nginx:1.24.0 --record=true
3         <none>
```

如果要回滚到指定版本，使用 **--to-revision** 参数

```bash
[root@k8s-master-1 ~]# kubectl rollout undo deploy nginx --to-revision=2
deployment.apps/nginx rolled back
```

## 六、扩缩容 Deployment

当业务量扩大，现有副本数已无法支撑业务系统是，可以对 Deployment 进行扩展

使用 **kubectl scale** 动态调整 Pod 的副本数，比如把 Pod 从 3 个扩容为 5 个，也可以从 5 个缩容为 3 个

- 查看现有副本数

```bash
[root@k8s-master-1 ~]# kubectl get pod 
NAME                     READY   STATUS    RESTARTS   AGE
nginx-5c97db8cdc-6vwfd   1/1     Running   0          2m14s
nginx-5c97db8cdc-m48rn   1/1     Running   0          2m15s
nginx-5c97db8cdc-ws9jz   1/1     Running   0          2m12s
```

- 使用 **kubectl scale** 进行扩容

```bash
[root@k8s-master-1 ~]# kubectl scale deploy nginx --replicas=5
deployment.apps/nginx scaled
```

- 再次查看，可以看到 Pod 变成了 5 个

```bash
[root@k8s-master-1 ~]# kubectl get pod 
NAME                     READY   STATUS    RESTARTS   AGE
nginx-5c97db8cdc-5bw25   1/1     Running   0          18s
nginx-5c97db8cdc-6vwfd   1/1     Running   0          5m27s
nginx-5c97db8cdc-bfsjf   1/1     Running   0          18s
nginx-5c97db8cdc-m48rn   1/1     Running   0          5m28s
nginx-5c97db8cdc-ws9jz   1/1     Running   0          5m25s
```

也可以使用 **kubectl edit** 命令修改 Deployment 进行扩缩容

```bash
[root@k8s-master-1 ~]# kubectl edit deploy nginx
deployment.apps/nginx edited
```

- 修改参数如下

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
...
spec:
  progressDeadlineSeconds: 600
  replicas: 3    # 修改 replicas 的值，就是对应要启动的副本数
  revisionHistoryLimit: 10
  ...
  template:
    ...
```

## 七、暂停和恢复 Deployment 更新

更新、扩容、缩容均是修改后立即触发更新，大多数情况下可能需要针对一个 Deployment 更改多处地方，而并不需要多次触发更新，此次可以使用 Deployment 暂停功能，临时禁用更新操作，对 Deployment 进行多次修改后在进行更新。这样做能够在暂停和恢复执行之间应用多个修补程序，而不会触发不必要的上线操作。

> 注意：不可以回滚处于暂停状态的 Deployment，除非先恢复其执行状态。

使用 **kubectl rollout pause** 命令即可 **暂停Deployment** 更新

```bash
[root@k8s-master-1 ~]# kubectl rollout pause deploy nginx
deployment.apps/nginx paused
```

 然后对 Deployment 进行相关更新镜像操作

```bash
[root@k8s-master-1 study]# kubectl set image deploy nginx nginx=repository.test.com:8444/kubernetes/nginx/nginx:1.24.0
deployment.apps/nginx image updated
```

对 Deployment 进行资源限制

```bash
[root@k8s-master-1 study]# kubectl set resources deploy nginx -c=nginx --limits=cpu=100m,memory=200Mi
deployment.apps/nginx resource requirements updated
```

通过 **rollout history** 可以看到没有进行新的更新

```bash
[root@k8s-master-1 study]# kubectl rollout history deploy nginx
deployment.apps/nginx 
REVISION  CHANGE-CAUSE
1         <none>
```

进行完最后一处配置更改后，使用 **kubectl rollout resume 恢复 Deployment 更新**

```bash
[root@k8s-master-1 study]# kubectl rollout resume deploy nginx
deployment.apps/nginx resumed
```

通过 rollout history 可以看到已经有了更新

```bash
[root@k8s-master-1 study]# kubectl rollout history deploy nginx
deployment.apps/nginx 
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
```

使用 **--revision** 查看最近一次版本的详细更新信息

```bash
[root@k8s-master-1 study]# kubectl rollout history deploy nginx --revision=2
deployment.apps/nginx with revision #2
Pod Template:
  Labels:       app=nginx
        pod-template-hash=6f7d5475bd
  Containers:
   nginx:
    Image:      repository.test.com:8444/kubernetes/nginx/nginx:1.24.0  # 可以看到镜像版本已经变成新的镜像
    Port:       <none>
    Host Port:  <none>
    Limits:  # 新增的limit配置也已经更新进来
      cpu:      100m
      memory:   200Mi
    Environment:        <none>
    Mounts:     <none>
  Volumes:      <none>
```

使用 describe 查看 Deployment 像详细信息

```bash
[root@k8s-master-1 study]# kubectl describe deploy nginx
Name:                   nginx
Namespace:              default
CreationTimestamp:      Tue, 30 May 2023 17:42:52 +0800
Labels:                 app=nginx
Annotations:            deployment.kubernetes.io/revision: 2
Selector:               app=nginx
Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=nginx
  Containers:
   nginx:
    Image:      repository.test.com:8444/kubernetes/nginx/nginx:1.24.0   # 可以看到镜像版本已经变成新的镜像
    Port:       <none>
    Host Port:  <none>
    Limits:   # 新增的limit配置也已经更新进来
      cpu:        100m
      memory:     200Mi
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   nginx-6f7d5475bd (3/3 replicas created)
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  4m3s   deployment-controller  Scaled up replica set nginx-64f5bcd79f to 3
  Normal  ScalingReplicaSet  3m19s  deployment-controller  Scaled up replica set nginx-6f7d5475bd to 1
  Normal  ScalingReplicaSet  3m16s  deployment-controller  Scaled down replica set nginx-64f5bcd79f to 2
  Normal  ScalingReplicaSet  3m16s  deployment-controller  Scaled up replica set nginx-6f7d5475bd to 2
  Normal  ScalingReplicaSet  3m13s  deployment-controller  Scaled down replica set nginx-64f5bcd79f to 1
  Normal  ScalingReplicaSet  3m13s  deployment-controller  Scaled up replica set nginx-6f7d5475bd to 3
  Normal  ScalingReplicaSet  3m10s  deployment-controller  Scaled down replica set nginx-64f5bcd79f to 0
```

## 八、Deployment 更新注意事项

- 查看 Deployment 的更新策略

```bash
[root@k8s-master-1 study]# kubectl get deploy nginx -o yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
spec:
  ...
  strategy:   # 更新策略
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    ...
```

**更新策略：**

- **.spec.strategy.type**：
  - **Recreate**：表示重建，先删除掉旧的 Pod， 在创建新的 Pod；
  - **RellingUpdate**：表示滚动更新，可以指定 maxSurge 和 maxUnavailable 来控制滚动更新过程；
- **.spec.strategy.rollingUpdate.maxSurge**：可以超过期望值的最大 Pod 数，可选字段，默认为 25%，可以设置成数字或百分比，如果 maxUnavailable 为 0，则该值不能为 0；
- **.spec.strategy.rollingUpdate.maxUnavailable**：指定在滚动更新时最大不可用的 Pod 数量，可选字段，默认时 25%，可以设置成数字或百分比，如果 maxSurge 为 0，则该值不能为 0；

如果使用 kubectl edit 命令修改 deployment文件，可以直接进行多次修改，无需暂停更新（最常用的方式）

**版本清理清理策略：**在默认情况下，**revision 只保留 10 个旧的 ReplicaSet** ，其余的将在后台进行垃圾回收，可以在 **.spec.revisionHistoryLimit 设置保留 ReplicaSet 的个数**。当设置为 0 时，不保留历史记录

**Ready 策略：**

**.spec.minReadySeconds**：可选参数，指定新创建的 Pod 应该在没有任何容器崩溃的情况下视为 **Ready（就绪）** 状态的最小秒数，默认为 0，即一旦被创建旧视为可用，通常和容器探针一起使用。

## 九、Deployment 状态

Deployment 的生命周期中会有许多状态，上线新的 ReplicaSet 期间可能处于 **Progressing（进行中）**，可能是 **Complete（已完成）**，也可能是 **Failed（失败）** 以至于无法继续进行。

### 进行中的 Deployment

执行下面的任务期间，Kubernetes 标记 Deployment 为 **进行中（Progressing）**:

- Deployment 创建新的 ReplicaSet；
- Deployment 正在为其最新的 ReplicaSet 扩容；
- Deployment 正在为其旧有的 ReplicaSet 缩容；
- 新的 Pod 已经就绪或者可用（就绪至少持续了 **MinReadySeconds** 秒）；

当上线过程进入 **Progressing** 状态时，Deployment 控制器会向 Deployment 的 **.status.conditions** 中添加包含下面属性的状况条目：

- **type: Progressing**
- **status: "True"**
- **reason：NewReplicaSetCreated | FoundNewReplicaSet | ReplicaSetUpdated**

可以使用 **kubectl rollout status** 监视 Deployment 的进度。

### 完成的 Deployment

当 Deployment 具有一下特征时，Kubernetes 将其标记为 **完成（Complete）**：

- 与 Deployment 关联的所有副本都已更新到指定的最新版本，这意味着之前请求的所有更新都已完成；
- 与 Deployment 关联的所有副本都可用；
- 未运行 Deployment 的旧副本；

当上线过程进入 “Complete” 状态时，Deployment 控制器会向 Deployment 的 **.status.conditions** 中添加包含下面属性的状况条目：

- **type: Progressing**
- **status: "True"**
- **reason: NewReplicaSetAvailable**

这一 **Progressing** 状况的状态值会持续为 **"True"**，直至新的上线动作被触发。即使副本的可用状态发生变化（进而影响 **Available** 状况），**Progressing** 状况的值也不会变化。

可以使用 **kubectl rollout status** 检查 Deployment 是否已完成，如果上线完成，**kubectl rollout status** 返回退出代码 0.

```bash
[root@k8s-master-1 ~]# kubectl rollout status deploy nginx-deploy

# 输出类似于：
deployment "nginx-deploy" successfully rolled out
```

### 失败的 Deployment

Deployment 可能会在尝试部署其最新的 ReplicaSet 受挫，一直处于未完成状态。造成此情况的一些可能因素如下：

- 配额（Quota）不足；
- 就绪探测（Readiness Probe）失败；
- 镜像拉取错误；
- 权限不足；
- 限制范围（Limit Ranges）问题；
- 应用程序运行时的配置错误

检测此状况的一种方式是在 Deployment 规约中指定截止时间参数：**（.spec.progressDeadlineSeconds)**。**.spec.progressDeadlineSeconds** 给出的是一个秒数值， Deployment 控制器在（通过 Deployment 状态）标示 Deployment 进展停滞之前，需要等待所给的时长。

以下 kubectl 命令设置规约中的 **progressDeadlineSeconds**，从而告知控制器在 10 分钟后报告 Deployment 的上线没有进展：

```bash
[root@k8s-master-1 ~]# kubectl patch deploy nginx-deploy -p '{"spec":{"progressDeadlineSeconds": 600}}'

# 输出类似于：
deployment.apps/nginx-deploy patched (no change)
```

超过截止时间后，Deployment 控制器将添加具有以下属性的 Deployment 状况到 Deployment 的 **.status.conditions** 中：

- **type: Progressing**
- **status: "False"**
- **reason: ProgressDeadlineExceeded**

这一状况也可能会比较早地失败，因而其状态值被设置为 "False"，其原因为 **ReplicaSetCreateError**。一旦 Deployment 上线完成，就不在考虑其期限。

> 除了报告 **Reason=ProgressDeadlineExceeded** 状态之外，Kubernetes 对已停止的 Deployment 不执行任何操作。更高级别的编排器可以利用这一设计并相应地采取行动。例如：将 Deployment 回滚到其以前的版本。
>
> 如果你暂停了某个 Deployment 上线，Kubernetes 不再根据指定的截止时间检查 Deployment 上线的进展。你可以在上线过程中间安全地暂停 Deployment 再恢复其执行，这样做不会导致超出最后时限的问题。

### 对失败的 Deployment 的操作

可应用于已完成的 Deployment 的所有操作也适用于失败的 Deployment。你可以对其执行扩缩容、回滚到以前的修订版本等操作，或者在需要对 Deployment 的 Pod 模版应用多项调整时，将 Deployment 暂停。

## 十、Deployment 清理策略

可以在 Deployment 中设置 **.spec.revisionHistoryLimit** 字段以指定保留此 Deployment 的多少个旧有 ReplicaSet。其余的 ReplicaSet 将在后台被垃圾回收。默认情况下，此值为 10。

**注意：将此字段设置为 0 将导致 Deployment 的所有历史记录被清空，因此 Deployment 将无法回滚。**

## 十一、Deployment 示例文件

```yaml
apiVersion: v1       #必填，版本号，例如v1
kind: Depolyment     #必填
metadata:       #必填，元数据
  name: string       #必填，Pod名称
  namespace: string    #必填，Pod所属的命名空间
  labels:      #自定义标签
    - name: string     #自定义标签名字<key: value>
  annotations:       #自定义注释列表
    - name: string
spec:         #必填，部署的详细定义
  selector: 
    matchLabels:
      name: string #必填，通过此标签匹配对应pod<key: value>
  replicas: number #必填，副本数量
  template: #必填，应用容器模版定义
    metadata: 
      labels: 
        name: string #必填，遇上面matchLabels的标签相同
    spec: 
      containers:      #必填，定义容器列表
      - name: string     #必填，容器名称
        image: string    #必填，容器的镜像名称
        imagePullPolicy: [Always | Never | IfNotPresent] #获取镜像的策略 Alawys表示下载镜像 IfnotPresent表示优先使用本地镜像，否则下载镜像，Nerver表示仅使用本地镜像
        command: [string]    #容器的启动命令列表，如不指定，使用打包时使用的启动命令
        args: [string]     #容器的启动命令参数列表
        workingDir: string     #选填，容器的工作目录
        env:       #容器运行前需设置的环境变量列表
        - name: string     #环境变量名称
          value: string    #环境变量的值
        ports:       #需要暴露的端口库号列表
        - name: string     #选填，端口号名称
          containerPort: int   #容器需要监听的端口号
          hostPort: int    #选填，容器所在主机需要监听的端口号，默认与Container相同
          protocol: string     #选填，端口协议，支持TCP和UDP，默认TCP
        resources:       #资源限制和请求的设置
          limits:      #资源限制的设置
            cpu: string    #Cpu的限制，单位为core数，将用于docker run --cpu-shares参数
            memory: string     #内存限制，单位可以为Mib/Gib，将用于docker run --memory参数
          requests:      #资源请求的设置
            cpu: string    #Cpu请求，容器启动的初始可用数量
            memory: string     #内存清楚，容器启动的初始可用数量
        volumeMounts:    #挂载到容器内部的存储卷配置
        - name: string     #引用pod定义的共享存储卷的名称，需用volumes[]部分定义的的卷名
          mountPath: string    #存储卷在容器内mount的绝对路径，应少于512字符
          readOnly: boolean    #是否为只读模式
        livenessProbe:     #对Pod内个容器健康检查的设置，当探测无响应几次后将自动重启该容器，检查方法有exec、httpGet和tcpSocket，对一个容器只需设置其中一种方法即可
          exec:      #对Pod容器内检查方式设置为exec方式
            command: [string]  #exec方式需要制定的命令或脚本
          httpGet:       #对Pod内个容器健康检查方法设置为HttpGet，需要制定Path、port
            path: string
            port: number
            host: string
            scheme: string
            HttpHeaders:
            - name: string
              value: string
          tcpSocket:     #对Pod内个容器健康检查方式设置为tcpSocket方式
            port: number
          initialDelaySeconds: 0  #容器启动完成后首次探测的时间，单位为秒
          timeoutSeconds: 0   #对容器健康检查探测等待响应的超时时间，单位秒，默认1秒
          periodSeconds: 0    #对容器监控检查的定期探测时间设置，单位秒，默认10秒一次
          successThreshold: 0
          failureThreshold: 0
          securityContext:
            privileged:false
        #Pod的重启策略，Always表示一旦不管以何种方式终止运行，kubelet都将重启，OnFailure表示只有Pod以非0退出码退出才重启，Nerver表示不再重启该Pod
        restartPolicy: [Always | Never | OnFailure]
        nodeSelector: obeject  #设置NodeSelector表示将该Pod调度到包含这个label的node上，以key：value的格式指定
        imagePullSecrets:    #Pull镜像时使用的secret名称，以key：secretkey格式指定
        - name: string
        hostNetwork:false      #是否使用主机网络模式，默认为false，如果设置为true，表示使用宿主机网络
        volumes:       #在该pod上定义共享存储卷列表
        - name: string     #共享存储卷名称 （volumes类型有很多种）
          emptyDir: {}     #类型为emtyDir的存储卷，与Pod同生命周期的一个临时目录。为空值
          hostPath: string     #类型为hostPath的存储卷，表示挂载Pod所在宿主机的目录
            path: string     #Pod所在宿主机的目录，将被用于同期中mount的目录
        - name: string     #共享存储卷名称
          secret:      #类型为secret的存储卷，挂载集群与定义的secre对象到容器内部
            scretname: string  
            items:     
            - key: string     #选择secrets定义的某个key
              path: string    #文件内容路径
        - name: string     #共享存储卷名称
          configMap:     #类型为configMap的存储卷，挂载预定义的configMap对象到容器内部
            name: string
            items:
            - key: string     #选择configmap定义的某个key
              path: string     #文件内容路径
        - name: string     #共享存储卷名称
          persistentVolumeClaim:
            claimName: string     #类型为PVC的持久化存储卷
```

