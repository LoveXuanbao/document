## 一、什么是 Service

Kubernetes 中 Service 是将运行在一个或一组 Pod 上的网络应用程序公开为网络服务的方法。

Kubernetes 中 Service 的一个关键目标是让你无需修改现有应用以使用某种不熟悉的服务发现机制。可以在 Pod 集合中运行代码，无论该代码是为云原生环境设计的，还是被容器化的老应用。都可以使用 Service 让一组 Pod 可在网络上访问，这样客户端就能与之交互。

Service API 是 Kubernetes 的组成部分，它是一种抽象，帮助将 Pod 集合在网络上公开出去。每个 Service 对象定义端点的一个逻辑集合（通常这些端点就是 Pod）以及如何访问到这些 Pod 的策略。

例如：考虑一个无状态的后端服务，其中运行 3 个副本（Replicas），这些副本是可以互换的；前端不需要关心它们调用的是哪个后端。即便构成后端集合的实际 Pod 可能会发生变化，前端客户端不应该也没必要知道这些。而且它们也不必亲自跟踪后端的状态变化。

Service 抽象使这种解藕成为可能；Service 所对应的 Pod 集合通常由定义的选择算符（标签）来确定。

## 二、Service 的类型

Kubernetes Service 类型允许指定所需要的 Service 类型，主要包括以下几种：

- **ClusterIP**：在集群内部使用，通过集群的内部 IP 公开 Service，选择该值时 Service 只能够在集群内部访问。这也是没有为服务显式指定 type 时使用的默认值。
- **NodePort**：通过每个节点上的 IP 和静态端口（NodePort）公开 Service。为了让 Service 可通过节点端口访问，Kubernetes 会为 Service 配置集群 IP 地址，相当于请求了 type: ClusterIP 的服务；通俗来说就时在所有安装了 kube-proxy 的节点上打开一个端口，此端口可以代理至后端 Pod，可以通过 NodePort 从集群外部访问集群内的服务，格式为 NodeIP:NodePort；
- **LoadBalancer**：使用云厂商提供的负载均衡器向外部公开 Service。Kubernetes 不直接提供负载均衡组件；成本较高；
- **ExternalName**：将服务映射到 `externalName` 字段的内容（例如：映射到主机名 `api.foo.bar.example`）。该映射将集群的 DNS 服务器配置为返回具有该外部主机名值的 CNAME 记录；通过返回定义的 CNAME 别名；集群不会为之创建任何类型的代理；需要 1.7 或更高版本的 kube-dns 支持。

## 三、ClusterIP 类型

此默认 Service 类型从集群中为此预留的 IP 地址池中分配一个 IP 地址；其他几种 Service 类型在 ClusterIP 类型的基础上进行构建。

如果定义的 Service 将 `.spec.clusterIP` 设置为 `"None"`，则 Kubernetes 不会为其分配 IP 地址，而是认为其是无头服务。

**选择自己的 IP 地址**

在创建 Service 的请求中，可以通过设置 `spce.clusterIP` 字段来指定自己的集群 IP 地址。比如，希望复用一个已存在的 DNS 条目，或者遗留系统已经配置了一个固定的 IP 且很难重新配置；所选择的 IP 地址必须是合法的 IPV4 或者 IPV6 地址，并且这个 IP 地址在 API 服务器上所配置的 `service-cluster-ip-range` CIDR 范围内。如果尝试创建一个带有非法 ClusterIP 地址值的 Service，API服务器会返回 HTTP 状态码 **422**，表示值不合法。

### 3.1、ClusterIP 示例文件

```yaml
apiVersion: v1    # 必须，API 版本
kind: Service     # 必须，资源类型
metadata:         # 必须，元数据定义
  name: nginx     # 必须，Service 名称
  namespace: default  # 必须，命名空间，尽量保持和调度器在同一命名空间下
spec:                 # 必须，具体的定义信息
  ports:              # 必须，Service 端口定义
  - name: nginx       # 端口名称
    port: 80          # Service 的端口
    protocol: TCP     # 网络协议，默认为 TCP
    targetPort: 80    # 后端应用的端口
  selector:           # 标签选择器，用来选择 Pod
    app: nginx        # Pod 的标签
  type: ClusterIP     # Service 的类型，默认为 ClusterIP
```

### 3.2、创建 Service

```bash
[root@k8s-master-1 ~]# kubectl create -f nginx-svc.yaml 
service/nginx created
```

### 3.3、查看 Service

```bash
[root@k8s-master-1 ~]# kubectl get svc
NAME             TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
kubernetes       ClusterIP   172.10.0.1       <none>        443/TCP        8d
nginx            ClusterIP   172.10.50.243    <none>        80/TCP         17m
```

### 3.4、访问 Cluster Service

```bash
[root@k8s-master-1 ~]# curl http://172.10.50.243
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

## 四、NodePort 类型

如果将 type 字段设置为 `NodePort`，则 Kubernetes 控制平面将在 `--service-node-port-range` 标志所指定的范围内分配端口（默认值：30000-32767）。每个节点将该端口（每个节点上的相同端口号）上的流量代理到 Service。Service 在其 `.spec.ports[*].nodePort` 字段中显示义分配的端口。

使用 NodePort 可以自由设置自己的负载均衡解决方案，配置 Kubernetes 不完全支持的环境，甚至直接公开一个或多个节点的 IP 地址。

对于 NodePort 服务，Kubernetes 额外分配一个端口（TCP、UDP 或 SCTP 以匹配 Service 的协议）。集群中的每个节点都将自己配置为监听所分配的端口，并将流量转发到与该 Service 关联的某个就绪端点（Pod）；通过使用合适的协议（例如 TCP）和适当的端口（分配给该 Service）连接到任何一个节点，就能够从集群外部访问 `type: NodePort` 服务。

**手动指定端口**

如果需要特定的端口号，可以在 nodePort 字段中指定一个有效的端口号。注意不要发生端口冲突。

**预留 NodePort 端口范围以避免发生冲突**

Kubernetes 1.29 新特性，为 NodePort 服务分配端口的策略既适用于自动分配的情况，也适用手动分配的场景。当某个用于希望创建一个使用特定端口的 NodePort 服务时，该目标端口可能与另一个已经被分配的端口冲突。为了避免这个问题，用于 NodePort 服务的端口范围被分为两段。动态端口分配默认使用较高的端口段，并且在较高的端口段耗尽时也可以使用较低的端口段。用户可以从较低端口段中分配端口，降低端口冲突的风险。

**为 NodePort 服务自定义 IP 地址配置**

用户可以配置集群中的节点使用特定的 IP 地址来支持 NodePort 服务。如果每个节点都连接到多个网络（例如：一个网络用于应用流量，另一个网络用于节点和控制节点之间的流量）。可以将 kube-proxy 的 `--nodeport-address` 标志或 `kube-proxy配置文件` 中的等效字段 `nodePortAddress` 设置为特定的 IP 段；此标志接受逗号分隔的 IP 段列表（例如 `10.0.0.0/8、192.0.1.0/25`），用来设置 IP 地址范围，kube-proxy 应该将其视为所在节点的本机地址。

例如：如果使用 `--nodeport-address=127.0.0.0/8` 标志启动 kube-proxy，则 kube-proxy 仅选择 NodePort 服务的本地回环接口。`--nodeport-address` 的默认值时一个空的列表。这意味着 kube-proxy 将认为所有可用网络接口都用于 NodePort 服务（这也与早期的 Kubernetes 版本兼容）。

> 说明：此 Service 的可见形式为 `<NodeIP>:spec.ports[*].nodePort` 以及 `.spec.clusterIP:spec.ports[*].port`。如果设置了 kube-proxy 的 `--nodeport-address` 标志或 kube-proxy 配置文件中的等效字段，则 `<NodeIP>` 将是一个被过滤的节点 IP 地址（或可能是多个 IP 地址）。

### 4.1、NodePort 指定端口示例文件

```yaml
apiVersion: v1    # 必须，API 版本
kind: Service     # 必须，资源类型
metadata:         # 必须，元数据定义
  name: nginx-nodeport     # 必须，Service 名称
  namespace: default  # 必须，命名空间，尽量保持和调度器在同一命名空间下
spec:                 # 必须，具体的定义信息
  ports:              # 必须，Service 端口定义
  - name: nginx       # 端口名称
    port: 80          # Service 的端口
    protocol: TCP     # 网络协议，默认为 TCP
    targetPort: 80    # 后端应用的端口
    nodePort: 30080
  selector:           # 标签选择器，用来选择 Pod
    app: nginx        # Pod 的标签
  type: NodePort     # Service 的类型，默认为 ClusterIP
```

### 4.2、创建及查看

```bash
[root@k8s-master-1 ~]# kubectl create -f nginx-svc-nodeport.yaml
service/nginx-nodeport created
[root@k8s-master-1 ~]# kubectl get svc
NAME             TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
kubernetes       ClusterIP   172.10.0.1       <none>        443/TCP        8d
nginx            ClusterIP   172.10.50.243    <none>        80/TCP         17m
nginx-nodeport   NodePort    172.10.104.233   <none>        80:30080/TCP   3s
```

### 4.3、通过 NodePort 访问

可以通过 `节点IP:NodePort端口` 来访问

```bash
# 获取第一台节点
[root@k8s-master-1 ～]# kubectl get nodes | head -2
NAME            STATUS   ROLES    AGE   VERSION
10.202.43.114   Ready    <none>   8d    v1.19.0

[root@k8s-master-1 ～]# curl http://10.202.43.114:30080
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

## 五、ExternalName 类型

ExternalName Service 是 Service 的特例，它没有 Selector，也没有定义任何端口的 Endpoint，它通过返回该外部服务的别名来提供服务。

例如：可以定义一个 Service，后端设置为一个外部域名，这样通过 Service 的名称即可访问到该域名。使用 nslookup 解析以下文件定义的 Service，集群的 DNS 服务将返回一个值为 `my.database.example.com` 的 CNAME 记录:

> Kubernetes DNS 服务器是唯一的一种能够访问 `ExternalName` 类型的 Service 的方式。

**示例文件**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: database
  namespace: default
spec:
  type: ExternalName
  externalName: my.database.example.com
```

**创建测试**

```bash
[root@k8s-master-1 ~]# kubectl get svc database
NAME       TYPE           CLUSTER-IP   EXTERNAL-IP               PORT(S)   AGE
database   ExternalName   <none>       my.database.example.com   <none>    8m37s

# 测试
curl database
ping database
```

当查找主机 `database.default.svc.cluster.local` 时，集群 DNS 服务返回 CNAME 记录，其值为 `my.database.example.com`。访问 database 的方式与访问其他 Service 到的方式相同，主要区别在于重定向发生在 DNS 级别，而不是通过代理或转发来完成。如果后来决定将数据库迁移到集群中，则可以启动其 Pod，添加适当的选择算符或端点并更改 Service 的 type。

> 说明：ExternalName 的服务接受 IPV4 地址字符串，但将该字符串视为由数字组成的 DNS 名称，而不是 IP 地址（然而，互联网不允许在 DNS 中使用此类名称）。类似于 IPV4 地址的外部名称无法被 DNS 服务器解析。
>
> 注意：针对 ExternalName 服务使用一些常见的协议，包括 HTTP 和 HTTPS，可能会有些问题。如果使用 ExternalName 服务，那么集群内客户端使用的主机名于 ExternalName 引用的名称不同。对于使用主机名的协议，这一差异可能会导致错误或意外响应。HTTP 请求将具有源服务器无法识别的 `Host:` 标头；TLS 服务器将无法提供与客户端连接的主机名匹配的证书。

## 六、Headless 类型

**Headless Service**，又称为无头服务，有时并不需要负载均衡，也不需要单独的 Service IP。遇到这种情况，可以通过显式设置集群 IP（spec.clusterIP）的值为 `"None"` 来创建**无头服务（Headless Service）**。

可以使用无法 Service 与其他服务发现机制交互，而不必绑定到 Kubernets 的实现。

无头 Service 不会获得集群 IP，kube-proxy 不会处理这类 Service，而且平台也不会为它们提供负载均衡或路由支持。取决于 Servcie 是否定义了选择算符，DNS 会以不同的方式被自动配置。

**带选择算符的服务**

对定义了选择算符的无头 Service，Kubernetes 控制节点在 Kubernetes API 中创建 EndpointSlice 对象，并且修改 DNS 配置返回 A 或 AAAA 记录（IPV4 或 IPV6 地址），这些记录直接指向 Service 的后端的 Pod 集合。

**无选择算符的服务**

对没有定义选择算符的无头 Service，控制节点不会创建 EndpointSlice 对象。然而 DNS 系统会执行以下操作之一：

- 对于 `type: ExternalName` Servie，查找和配置其 CNAME 记录；
- 对所有其他类型的 Service，针对 Service 的就绪端点的所有 IP 地址，查找和配置 DNS A/AAAA 记录：
    - 对于 IPV4 端点，DNS 系统创建 A 记录；
    - 对于 IPV6 端点，DNS 系统创建 AAAA 记录/

> 当定义无选择算符的无头 Service 时，`port` 必须于 `targetPort` 匹配。

## 七、LoadBalancer 类型

在使用支持外部负载均衡器的云平台时，如果将 type 设置为 `"LoadBalancer"` ，则平台会为 Service 提供负载均衡器。负载均衡器的实际创建过程时异步进行的，关于所制备的负载均衡器的信息将会通过 Service 的 `status.loadBalancer` 字段公开出来。

来自外部负载均衡器的流量将被直接重定向到后端各个 Pod 上，云平台决定如果进行负载平衡。要实现 LoadBalancer 的服务，Kubernetes 通常首先进行与请求 NodePort 服务类似的更改。cloud-controller-manager 组件随后配置外部负载均衡器，以将流量转发到所分配的节点端口。

可以将负载均衡 Service 配置为忽略分配节点端口，前提是云平台实现该功能。

某些云平台允许设置 `loadBanlancerIP` 。这时，平台将使用用户指定的 `loadBalancerIP` 来创建负载均衡器。如果没有设置该字段，平台将会给负载均衡器分配一个临时 IP。如果设置了，但是云平台并不支持这一特性，所设置的 loadBalancerIP 值将会被忽略。

> 说明：针对 Service 的 `.spec.loadBalancerIP` 字段已经在 Kubernetes v1.24 中被弃用。



## 八、Service 代理 k8s 外部服务

**使用场景**

- 希望在生产环境中使用某个固定的名称而非 IP 地址访问外部的中间件服务；
- 希望 Service 指向另一个 NameSpace 中或其他集群中的服务；
- 正在将工作负载转移到 Kubernets集群，但是一部分服务仍运行在 Kubernetes 集群之外的；

如果需要使用 Service 代理外部服务，需要手动创建 Service 和 Endpoints；只要 Endpoints 名称与 Service 的名称一致它们就会自动建立连接

### 8.1、Service 示例文件

```yaml
apiVersion: v1
kind: Service
metadata:
  name: minio
  namespace: default
spec:
  ports:
  - name: http
    port: 9000
    protocol: TCP
    targetPort: 9000
  type: CLusterIP
```

### 8.2、Endpoints 示例文件

```yaml
apiVersion: v1         # API版本
kind: Endpoints        # 资源类型
metadata:              # 元数据信息
  name: minio          # endpoints名称，必须与 service 名称一致
  namespace: default   # 命名空间
subsets:               # 后端服务定义
- addresses:           # 后端服务的 IP 定义，列表可以写多个 
  -ip: 192.168.1.101
  -ip: 192.168.1.102
  -ip: 192.168.1.103
  -ip: 192.168.1.104
  ports:               # 后端服务的端口定义
  - name: http         # 名称必须与 Service 的端口定义一致
    port: 9000         # 端口
    protocol: TCP      # 协议
```

### 8.3、创建后验证

```bash
# 查看 service
[root@k8s-master-1 test]# kubectl get svc minio
NAME    TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
minio   ClusterIP   172.10.225.65   <none>        9000/TCP   8s

# 查看 endpoints
[root@k8s-master-1 test]# kubectl get ep minio
NAME    ENDPOINTS                                                              AGE
minio   192.168.1.101:9000,192.168.1.102:9000,192.168.1.103:9000 + 1 more...   14s
#访问测试
[root@k8s-master-1 ～]# curl http://172.10.225.65:9000 
<?xml version="1.0" encoding="UTF-8"?>
```

## 九、服务发现

对于集群内运行的客户端，Kubernetes 支持两种主要的服务发现模式：环境变量和 DNS。

### 9.1、环境变量

当 Pod 运行在某 Node 上时，kubelet 会在其中为每个活跃的 Service 添加一组环境变量。kubelet 会添加环境变量 `{SVCNAME}_SERVICE+HOST` 和 `{SVCNAME}_SERVICE_PORT`。这里 Service 的名称被转为大写字母，横线被转换成下划线。

例如：一个 Service `redis-primary` 公开了 TCP 端口 6379，同时被分配了集群 IP 地址 10.0.0.11。这个 Service 生成的环境变量如下：

```bash
REDIS_PRIMARY_SERVICE_HOST=10.0.0.11
REDIS_PRIMARY_SERVICE_PORT=6379
REDIS_PRIMARY_PORT=tcp://10.0.0.11:6379
REDIS_PRIMARY_PORT_6379_TCP=tcp://10.0.0.11:6379
REDIS_PRIMARY_PORT_6379_PROTO=tcp
REDIS_PRIMARY_PORT_6379_PORT=6379
REDIS_PRIMARY_6379_TCP_ADDR=10.0.0.11
```

> 说明：当 Pod 需要访问某 Service，并且在使用环境变量方法将端口和集群 IP 发布到客户端 Pod 时，必须在客户端 Pod 出现之前创建该 Service。否则，这些客户端 Pod 中将不会出现对应的环境变量。如果仅使用 DNS 来发现 Service 的集群 IP，则无需担心此顺序问题。

### 9.2、DNS

能够感知集群的 DNS 服务器（例如 CoreDNS）会监视 Kubernetes API 中的新 Service，并为每个 Service 创建一组 DNS 记录。如果在整个集群中都启用了 DNS，则所有 Pod 都应该能够通过 DNS 名称自动解析 Service。

例如：如果在 Kubernetes 命名空间 `test` 中有一个名为 `my-service` 的 Service，则控制节点和 DNS 服务共同为 `my-service.test` 生成 DNS 记录。命名空间 `test` 中的 Pod 应该能够通过按名检索 `my-service` 来找到服务（`my-service.test` 也可以）。

其他命名空间中的 Pod 必须将名称限定为 `my-service.test` 才能够找到服务；这些名称将解析为分配给 Service 的集群 IP。

Kubernetes 还支持命名端口的 DNS SRV（Service）记录。如果 Service `my-service.test` 具有名为 http 端口，且协议设置为 TCP，则可以用 `_http._tcp.my-service.test` 执行 DNS SRV 查询以发现 http 的端口号以及 IP 地址。
