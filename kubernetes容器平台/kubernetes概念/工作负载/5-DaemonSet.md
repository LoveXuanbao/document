## 一、什么是 DaemonSet

官方网址：https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/daemonset/

DaemonSet（守护进程集，缩写为 ds）和守护进程类似，它在符合匹配条件的节点上均部署一个 Pod。当有新节点加入集群时，也会为它们新增一个 Pod，当节点从集群中移除时，这些 Pod 也会被回收，删除 DaemonSet 将会删除它创建的所有 Pod。

DaemonSet 的一些典型用法：

- 运行集群存储 DaemonSet（守护进程），例如在每个节点上运行 Glusterd、Ceph等；
- 在每个节点运行日志收集 DaemonSet，例如：Fluentd、Logstash等；
- 在每个节点运行监控 DaemonSet，例如：Prometheus Node Exporter、Collectd、Datadog 代理、New Relic 代理或 Ganglia gmond 等；

一种简单的用法是为每种类型的守护进程在所有节点上都启动一个 DaemonSet。一个稍微复杂的用法是为同一种守护进程部署多个 DaemonSet；每个具有不同的标志，并且对不同硬件类型具有不同的内存、CPU 要求。

## 二、DamonSet 资源文件

`vim nginx-ds.yaml`

```yaml
apiVersion: apps/v1  #必须，api版本
kind: DaemonSet      #必须，api类型
metadata:            #必须，元数据信息
  labels:            #可选，标签
    app: nginx-ds
  name: nginx-ds      #必须，名称
spec:                #控制器配置信息
  revisionHistoryLimit: 10
  updateStrategy:    #更新策略
    rollingUpdate: 
      maxUnavailable: 1  #最大不可用1
    type: RollingUpdate #默认滚动更新，还有其他策略为Ondelete
  selector:          #选择器配置
    matchLabels:
      app: nginx-ds
  template:          #以下为pod配置
    metadata:
      creationTimestamp: null
      labels:
        app: nginx-ds
    spec:
      tolerations:
      # 这些容忍设置时为了让该守护进程在控制节点上运行，如果不需要，可以删除
      - key: node-role.kubernetes.io/master
        effect: NoScheduler
      containers:
      - name: nginx-ds
        image: repository.test.com:8444/kubernetes/nginx/nginx:1.7.9
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 80
```

### 2.1、创建 DaemonSet

```bash
[root@k8s-master-1 ~]# kubectl create -f nginx-ds.yaml 
daemonset.apps/nginx-ds created
```

- 查看 DaemonSet

```bash
[root@k8s-master-1 ~]# kubectl get daemonset
NAME       DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
nginx-ds   5         5         5       5            5           <none>          3m46s
```

- 查看 Pod

```bash
[root@k8s-master-1 ~]# kubectl get pod -o wide
NAME             READY   STATUS    RESTARTS   AGE     IP              NODE            NOMINATED NODE   READINESS GATES
nginx-ds-6ghj5   1/1     Running   0          4m55s   172.16.217.5    10.202.43.55    <none>           <none>
nginx-ds-6ksmz   1/1     Running   0          4m55s   172.16.110.12   10.202.43.46    <none>           <none>
nginx-ds-9hzdl   1/1     Running   0          4m55s   172.16.22.15    10.202.43.114   <none>           <none>
nginx-ds-fwpdr   1/1     Running   0          4m55s   172.16.160.6    10.202.43.56    <none>           <none>
nginx-ds-qf7gc   1/1     Running   0          4m55s   172.16.49.14    10.202.43.191   <none>           <none>
```

## 三、DaemonSet 如何调度

![image-20240313221827455](https://typora-1306014148.cos.ap-nanjing.myqcloud.com/markdown/image-20240313221827455.png)

DaemonSet 可用于确保所有符合条件的节点都运行该 Pod 的一个副本。DaemonSet 控制器为每个符合条件的节点创建一个 Pod，并添加 Pod 的 `spec.affinity.nodeAffinity` 字段以匹配目标主机。Pod 被创建之后，默认的调度程序通常通过设置 `.spec.nodeName` 字段来接管 Pod 并将 Pod 绑定到目标主机。如果新的 Pod 无法放在节点上，则默认的调度程序可能会根据新 Pod 的优先级抢占（驱逐）某些现存的 Pod。

> 说明：当 DaemonSet 中的 Pod 必须运行在每个节点上时，通常需要将 DaemonSet 的 `.spec.template.spec.priorityClassName` 设置为具有更高优先级的 **PriorityClass** ，以确保可以完成驱逐。

用户可以通过设置 DaemonSet 的 `.spec.template.spec.schedulerName` 字段，可以为 DaemonSet 的 Pod 指定不同的调度程序。

当评估符合条件的节点时，原本在 `.spec.template.spec.affinity.nodeAffinity` 字段上指定的节点亲和性将由 DaemonSet 控制器进行考量，但在创建的 Pod 上会被替换为符合条件的节点名称匹配和节点亲和性。

`scheduleDaemonSetPods` 允许使用默认调度器而不是 DaemonSet 控制器来调度这些 DaemonSet，方法是将 `NodeAffinity` 条件而不是 `.spec.nodeName` 条件添加到这些 DaemonSet Pod。默认调度器接下来将 Pod 绑定到目标主机。如果 DaemonSet Pod 的节点亲和性配置已存在，则被替换（原始的节点亲和性配置在选择目标主机之前被考虑）。DaemonSet 控制器仅在创建或修改 DaemonSet Pod 时执行这些操作，并不会更改 DaemonSet 的 `spec.template`。

>  DaemonSet 调度不同于普通的 Pod 调度，所以没有用默认的 Kubernetes Scheduler 进行调度，而是通过专有的 DaemonSet Controller 进行调度。但是随着 Kubernetes 版本的改进和调度特性不断丰富，产生了一些难以解决的矛盾，最主要的两个矛盾如下：
>
> - 普通的 Pod 是在 Pending 状态触发调度并被实例化的，DaemonSet Controller 并不是在这个状态调度 Pod 的，这种不一致容器误导和迷惑用户；
> - Pod 优先级调度是被 Kubernetes Scheduler 执行的，而 DaemonSet Controller 并没有考虑到 Pod 优先级调度的问题，也产生了不一致的结果；
>
> 从 Kubernetes 1.18 开始，DaemonSet 的调度默认切换到 Kubernetes Scheduler 进行，从而一劳永逸地解决了以上问题及未来可能的新问题。因为默认切换到了 Kubernetes Scheduler 统一调度 Pod，因此 DaemonSet 也能正确处理 Taints 和 Tolerations 的问题。

## 四、DaemonSet 更新

如果节点的标签被修改，DamonSet 将立刻向新匹配的节点上添加 Pod，同时删除不匹配的节点上的 Pod。

可以修改 DaemonSet 创建的 Pod，不过并非 Pod 的所有字段都可更新。下次当某节点（即使具有相同的名称）被创建时，DaemonSet 控制器还会使用最初的模版。

可以删除一个 DaemonSet。如果使用 kubectl 并指定 **--cascade=orphan** 选项，则 Pod 将被保留在节点上。接下来如果创建使用相同选择算符的新 DaemonSet，新的 DaemonSet 会收养已有的 Pod。如果有 Pod 需要被替换，DamonSet 会根据其 **uodateStrategy** 来替换。

### 4.1、DaemonSet 更新策略

DamonSet 有两种更新策略：

- **OnDelete**：使用 OnDelete 更新策略时，在更新 DaemonSet 模版后，只有当你手动删除老的 DaemonSet Pod 之后，新的 DaemonSet Pod 才会被自动创建。跟 Kubernetes 1.6 以前的版本类似。
- **RollingUpdate**：这是默认的更新策略。使用 RollingUpdate 更新策略时，在更新 DaemonSet 模版后，老的 DaemonSet Pod 将被终止，并且将以受控方式自动创建新的 DaemonSet Pod。更新期间，最多只能有 DaemonSet 的一个 Pod 运行与每个节点上。

### 4.2、RollingUpdate 更新演示

**更新容器镜像**

```bash
[root@k8s-master-1 ~]# kubectl set image ds nginx-ds nginx-ds=repository.test.com:8444/kubernetes/nginx/nginx:1.24.0 --record=true

# 查看滚动更新的状态，在更新时会依次对每个节点上的 Pod 进行更新
[root@k8s-master-1 test]# kubectl get pod -o wide -w
NAME             READY   STATUS        RESTARTS   AGE    IP              NODE            NOMINATED NODE   READINESS GATES
nginx-ds-lzffx   1/1     Running       0          67s    172.16.22.17    10.202.43.114   <none>           <none>
nginx-ds-mqnww   1/1     Running       0          98s    172.16.160.8    10.202.43.56    <none>           <none>
nginx-ds-pvjdg   1/1     Running       0          77s    172.16.49.16    10.202.43.191   <none>           <none>
nginx-ds-slv2f   0/1     Terminating   0          87s    172.16.110.14   10.202.43.46    <none>           <none>
nginx-ds-z4kpb   1/1     Running       0          108s   172.16.217.7    10.202.43.55    <none>           <none>
nginx-ds-6fggj   0/1     Pending       0          0s     <none>          <none>          <none>           <none>
nginx-ds-6fggj   0/1     Pending       0          0s     <none>          10.202.43.46    <none>           <none>
nginx-ds-6fggj   0/1     ContainerCreating   0          0s     <none>          10.202.43.46    <none>           <none>
nginx-ds-6fggj   1/1     Running             0          3s     172.16.110.15   10.202.43.46    <none>           <none>
nginx-ds-lzffx   0/1     Terminating         0          74s    172.16.22.17    10.202.43.114   <none>           <none>
nginx-ds-2gptn   0/1     Pending             0          0s     <none>          <none>          <none>           <none>
nginx-ds-2gptn   0/1     Pending             0          0s     <none>          10.202.43.114   <none>           <none>
nginx-ds-2gptn   0/1     ContainerCreating   0          0s     <none>          10.202.43.114   <none>           <none>
nginx-ds-2gptn   1/1     Running             0          2s     172.16.22.18    10.202.43.114   <none>           <none>
nginx-ds-pvjdg   0/1     Terminating         0          88s    172.16.49.16    10.202.43.191   <none>           <none>
nginx-ds-vls2s   0/1     Pending             0          0s     <none>          <none>          <none>           <none>
nginx-ds-vls2s   0/1     Pending             0          0s     <none>          10.202.43.191   <none>           <none>
nginx-ds-vls2s   0/1     ContainerCreating   0          0s     <none>          10.202.43.191   <none>           <none>
nginx-ds-vls2s   1/1     Running             0          2s     172.16.49.17    10.202.43.191   <none>           <none>
nginx-ds-z4kpb   0/1     Terminating         0          2m10s   172.16.217.7    10.202.43.55    <none>           <none>
nginx-ds-2t5xg   0/1     Pending             0          0s      <none>          <none>          <none>           <none>
nginx-ds-2t5xg   0/1     Pending             0          0s      <none>          10.202.43.55    <none>           <none>
nginx-ds-2t5xg   0/1     ContainerCreating   0          0s      <none>          10.202.43.55    <none>           <none>
nginx-ds-2t5xg   1/1     Running             0          2s      172.16.217.8    10.202.43.55    <none>           <none>
nginx-ds-mqnww   0/1     Terminating         0          2m10s   172.16.160.8    10.202.43.56    <none>           <none>
nginx-ds-fhrp4   0/1     Pending             0          0s      <none>          <none>          <none>           <none>
nginx-ds-fhrp4   0/1     Pending             0          0s      <none>          10.202.43.56    <none>           <none>
nginx-ds-fhrp4   0/1     ContainerCreating   0          0s      <none>          10.202.43.56    <none>           <none>
nginx-ds-fhrp4   1/1     Running             0          2s      172.16.160.9    10.202.43.56    <none>           <none>
```

**监视滚动更新状态**

```bash
[root@k8s-master-1 ~]# kubectl rollout status ds nginx-ds
Waiting for daemon set "nginx-ds" rollout to finish: 0 out of 5 new pods have been updated...
Waiting for daemon set "nginx-ds" rollout to finish: 0 out of 5 new pods have been updated...
Waiting for daemon set "nginx-ds" rollout to finish: 0 out of 5 new pods have been updated...
Waiting for daemon set "nginx-ds" rollout to finish: 1 out of 5 new pods have been updated...
Waiting for daemon set "nginx-ds" rollout to finish: 1 out of 5 new pods have been updated...
Waiting for daemon set "nginx-ds" rollout to finish: 1 out of 5 new pods have been updated...
Waiting for daemon set "nginx-ds" rollout to finish: 2 out of 5 new pods have been updated...
Waiting for daemon set "nginx-ds" rollout to finish: 2 out of 5 new pods have been updated...
Waiting for daemon set "nginx-ds" rollout to finish: 2 out of 5 new pods have been updated...
Waiting for daemon set "nginx-ds" rollout to finish: 3 out of 5 new pods have been updated...
Waiting for daemon set "nginx-ds" rollout to finish: 3 out of 5 new pods have been updated...
Waiting for daemon set "nginx-ds" rollout to finish: 3 out of 5 new pods have been updated...
Waiting for daemon set "nginx-ds" rollout to finish: 4 out of 5 new pods have been updated...
Waiting for daemon set "nginx-ds" rollout to finish: 4 out of 5 new pods have been updated...
Waiting for daemon set "nginx-ds" rollout to finish: 4 out of 5 new pods have been updated...
Waiting for daemon set "nginx-ds" rollout to finish: 4 of 5 updated pods are available...
daemon set "nginx-ds" successfully rolled out   # 如果间隔过长，可能会看不到上面的更新过程，但是只要出现此日志代表滚动更新完毕
```



## 五、DaemonSet 回滚

- 查看所有修订版本

```bash
[root@k8s-master-1 test]# kubectl rollout history daemonset nginx-ds
daemonset.apps/nginx-ds 
REVISION  CHANGE-CAUSE
1         <none>
2         kubectl set image ds nginx-ds nginx-ds=repository.test.com:8444/kubernetes/nginx/nginx:1.24.0 --record=true
```

- 回滚

```bash
# 回滚到指定版本
kubectl rollout undo daemonset <daemonset-name> --to-revision=<revision>
# eg:
kubectl rollout undo daemonset nginx-ds --to-revision=1

# 回滚到上一个版本
kubectl rollout undo daemonset <daemonset-name>
# eg:
kubectl rollout undo daemonset nginx-ds
```

## 六、DaemonSet 修订版本

在 **kubectl rollout history** 步骤中，获得一个修订版本列表，每个修订版本都存储在名为 **ControllerRevision** 的资源中。

要查看每个修订版本中保存的的内容，可以找到 DaemonSet 修订版定的原生资源：

```bash
kubectl get controllerrevision -l <daemonset-selector-key>=<daemonset-selector-value>

# eg:
[root@k8s-master-1 ~]# kubectl get controllerrevision -l app=nginx-ds
NAME                  CONTROLLER                REVISION   AGE
nginx-ds-55c7dcf7bc   daemonset.apps/nginx-ds   2          14m
nginx-ds-7874f9c78c   daemonset.apps/nginx-ds   3          15m
```

每个 ControllerRevision 中存储了相应的 DaemonSet 版本的注解和模版。

**kubectl rollout undo** 选择特定的 **ControllerRevision** ，并用 ControllerRevision 中存储的模版代替 DaemonSet 的模版，**kubectl rollout undo** 相当于通过其他命令（如：kubectl edit 或 kubectl apply）将 DaemonSet 模版更新至先前的版本。

> 注意：DaemonSet 修订版定只会正向变化。也就是说，回滚完成后，所回滚到的 ControllerRevision 版本号（`.revision 字段`）会增加。例如，如果在系统中有版本 1 和版本 2，并从版本 2 回滚到版本 1，带有 `.revision: 1` 的 ControllerRevision 将变为 `.revision: 3`
