## 一、标签和标签选择器

官方网址：https://kubernetes.io/zh-cn/docs/concepts/overview/working-with-objects/labels/#label-selectors

## 二、Label 是什么

#### label介绍

大多数情况下，我们创建的 Pod 副本会被调度到集群中的任何一个可用节点上，而不会关心具体会调度到哪个节点。不过某些情况存在一种需求：希望某种pod的副本全部在指定的一个或这一些节点上运行，比如希望将 MySQL 数据库调度到一个具有 SSD 磁盘的目标节点上。

这时候我们就需要通过 Kubernetes 的 `label`（标签）来实现这个功能。

**Labels（标签）**是 Kubernetes 的一个核心概念，是附加到 Kubernetes对象（比如 Pod）上的 **key=value** 的键值对；一个 label 是一个 `key=value` 的键值对，其中 `key` 与 `value` 都是用户自定义的。`label`可以添加到各种资源对象上，如`Node`、`Pod`、`Service`、`RC`、`Deployment`等。label 通常在资源对象定义时确定，也可以在对象创建后动态添加和删除，通常给指定的资源对象捆绑一个或多个不同的 label 来实现多维度的资源分组管理功能，以便灵活、方便的进行资源分配、调度、配置、部署等管理工作。例如：部署不同版本的应用到不同的环境中，以及监控、分析应用（日志记录、监控、告警）等。

标签旨在用于指定对用户有意义且相关的对象的标识属性，但不直接对核心系统有语义含义。标签可以用于组织和选择对象的子集。标签可以在创建时附加到对象，随后可以随时添加和修改。每个对象都可以定义一组键/值标签。每个键对于给定对象必须是唯一的。可以对 Kubernetes 的一些对象，如 Pod 和节点进行“分组”，用于区分同样的资源不同的分组；但并不对资源工作参数有任何影响。

- k8s特色的管理方式，便于分类管理资源对象
- 一个标签可以对应多个资源，一个资源也可以有多个标签。
- 一个资源拥有多个标签，可以实现不同维度的管理
- 一个 label 是 key=value 的键值对

标签使用户能够以松散耦合的方式将它们自己的组织结构映射到系统对象，而无需客户端存储这些映射。

**标签**是键值对，有效的标签键有两个段：可选的前缀和名称，用斜杠 `/` 分割。名称段是必须的，必须小于等于 63 个字符，以字母数字字符 `[a-z0-9A-Z]` 开头和结尾，带有破折号 `-`  、下划线 `_` 、点 `.` 和之间的字母数字；前缀是可选的，如果指定，前缀必须是 DNS 子域，由点 `.` 分割的一系列 DNS 标签。总共不超过 253 个字符，后跟斜杠 `/`。

> `kubernetes.io/` 和 `k8s.io/` 前缀是为 Kubernetes 核心组件保留的。

**有效标签值：**

- 必须为 63 个字符或更少（可以为空）；
- 除非标签值为空，否则必须以字母数字字符 `[a-z0-9A-Z]` 开头和结尾；
- 包含破折号 `-` 、下划线 `_` 、点 `.` 和字母或数字；

**常用的label实例如下：**

- 版本标签：`release: stable` 和 `release: canary`
- 环境标签：`environment: dev`、`environment: qa` 和 `environment: production`
- 架构标签：`tier: frontend`、`tier: backend` 和 `tier: middleware`
- 分区标签：`partition: customerA`和`partition: customerB`
- 质量管控标签：`track: daily` 和 `track: weekly`

## 三、标签的使用

### 3.1、查看标签

查看标签只要在 get 资源最后加上 `--show-lables` 命令即可

```bash
# 查看 Node 标签
[root@k8s-master-1 ~]# kubectl get nodes --show-labels
NAME            STATUS   ROLES    AGE    VERSION   LABELS
10.202.43.114   Ready    <none>   7d4h   v1.19.0   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=10.202.43.114,kubernetes.io/os=linux,node.kubernetes.io/node=
10.202.43.191   Ready    <none>   7d4h   v1.19.0   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=10.202.43.191,kubernetes.io/os=linux,node.kubernetes.io/node=
10.202.43.46    Ready    <none>   7d4h   v1.19.0   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=10.202.43.46,kubernetes.io/os=linux,node.kubernetes.io/node=
10.202.43.55    Ready    <none>   7d4h   v1.19.0   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=10.202.43.55,kubernetes.io/os=linux,node.kubernetes.io/node=
10.202.43.56    Ready    <none>   7d4h   v1.19.0   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=10.202.43.56,kubernetes.io/os=linux,node.kubernetes.io/node=

# 查看 Deployment 标签
[root@k8s-master-1 ～]# kubectl get deploy --show-labels
NAME           READY   UP-TO-DATE   AVAILABLE   AGE   LABELS
nginx-deploy   1/1     1            1           59m   app=nginx

# 查看 Pod 标签
[root@k8s-master-1 ～]# kubectl get pod --show-labels
NAME                           READY   STATUS    RESTARTS   AGE   LABELS
nginx-deploy-c4df7c787-z9vq6   1/1     Running   0          60m   app=nginx,pod-template-hash=c4df7c787
```

如果知道标签想查看对应的资源，可以使用 `-l key=value`

```bash
# 查看包含标签 app=nginx 的 Deployment 资源，其他资源用法类似
[root@k8s-master-1 ～]# kubectl get deploy -l app=nginx
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deploy   1/1     1            1           65m
```

### 3.2、添加label

大多数的 Kubernetes 中的资源都是可以进行打标签的，命令是 `kubectl label`

增加语法：`kubectl label nodes <node-name> <label-name>=<label-value>`

```bash
# 给指定的资源添加 label
[root@k8s-master-1 ~]# kubectl label deployment nginx-deploy address=beijing

# 查看是否添加成功
[root@k8s-master-1 ～]# kubectl get deploy --show-labels
NAME           READY   UP-TO-DATE   AVAILABLE   AGE   LABELS
nginx-deploy   1/1     1            1           65m   address=beijing,app=nginx

# 给某一类型的所有资源打标签
[root@k8s-master-1 ~]# kubectl label nodes --all role=test 
node/10.202.43.114 labeled
node/10.202.43.191 labeled
node/10.202.43.46 labeled
node/10.202.43.55 labeled
node/10.202.43.56 labeled

# 查看所有包含 role=test 的 Node 节点
[root@k8s-master-1 ~]# kubectl get nodes -l role=test
NAME            STATUS   ROLES    AGE    VERSION
10.202.43.114   Ready    <none>   7d4h   v1.19.0
10.202.43.191   Ready    <none>   7d4h   v1.19.0
10.202.43.46    Ready    <none>   7d4h   v1.19.0
10.202.43.55    Ready    <none>   7d4h   v1.19.0
10.202.43.56    Ready    <none>   7d4h   v1.19.0
```

### 3.3、修改标签

在原有增加标签基础上，添加 `--overwrite`参数

修改语法：`kubectl label nodes <node-name> <label-name>=<label-value> --overwrite`

> 注意：修改的时候需要添加`--overwrite`参数，不然会报错

```bash
# 修改 10.202.43.114 节点标签 role=demo
[root@k8s-master-1 test]# kubectl label nodes 10.202.43.114 role=demo --overwrite
node/10.202.43.114 labeled

# 修改所有 role=xxx 的标签为 role=production
[root@k8s-master-1 ~]# kubectl label nodes --all role=production --overwrite
node/10.202.43.114 labeled
node/10.202.43.191 labeled
node/10.202.43.46 labeled
node/10.202.43.55 labeled
node/10.202.43.56 labeled
```

### 3.4、删除标签

删除 label，在命令行最后执行 label 的 key-name 并于一个减号相连

删除语法：`kubectl label nodes <node-name> <label-name>-`

```bash
# 删除指定资源的标签
[root@k8s-master-1 test]# kubectl label nodes 10.202.43.114 role-
node/10.202.43.114 labeled

[root@k8s-master-1 test]# kubectl label deployment.apps/nginx-deploy address-
deployment.apps/nginx-deploy labeled

# 删除所有 Node 节点上的 role 标签
[root@k8s-master-1 test]# kubectl label nodes --all role-
label "role" not found.
node/10.202.43.114 not labeled
node/10.202.43.191 labeled
node/10.202.43.46 labeled
node/10.202.43.55 labeled
node/10.202.43.56 labeled
```

### 3.5、利用标签查找资源

`kubectl get` 可以打印资源列表，并且可以使用 `-l` 参数利用 label 进行资源筛选

- ==、=：等于
- !=：不等于
- in：包含
- notin：不包含

```bash
# 单个条件筛选
kubectl get pod -l app=nginx
kubectl get pod -l app!=nginx

# 多个条件筛选 and 与
kubectl get pod -l app=nginx,app=nginx-1

# 多个条件筛选 in 
kubectl get pod -l 'app in(nginx,nginx-1)'

# 多个条件筛选 notin
kubectl get pod -l 'app notin(nginx,nginx-1)'
```



**Selector（标签选择器）**：可以通过根据资源的标签查询出精确的对象信息；