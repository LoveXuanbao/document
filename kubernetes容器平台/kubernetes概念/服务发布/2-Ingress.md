## 一、Ingress 是什么

官方网址：https://kubernetes.io/zh-cn/docs/concepts/services-networking/ingress/

根据 Service 的概念说明，我们知道 Service 的表现形式为 IP 地址和端口号（ClusterIP:Port），即工作在 TCP/IP 层。而对于基于 HTTP 的服务来说，不同的 URL 地址经常对应到不同的后端服务或者虚拟服务器（Virtual Host），这些应用层的转发机制仅通过 Kubernetes 的 Service 机制是无法实现的。Kubernetes 从 1.1 版本开始引入 Ingress 资源对象，用于将 Kubernetes 集群外的客户端请求路由到集群内部的服务上，同时提供 7 层（HTTP 和 HTTPS）路由功能。Ingress 在 Kubernetes 1.19版本时达到 v1 稳定版本。

Kubernetes 使用了一个 Ingress 策略定义和一个具体提供转发服务的 Ingress Controller，两者结合，实现了基于灵活 Ingress 策略定义的服务路有功能。如果是对 Kubernetes 集群外部的客户端提供服务，那么 Ingress Controller 实现的是类似于边缘路由器（Edge Router）的功能。需要注意的是，Ingress 只能以 HTTP 和 HTTPS 提供服务，对于使用其他网络协议的服务，可以通过设置 Service 的类型（type）为 NodePort 或 LoadBalancer 对集群外部的客户端提供服务。

使用 Ingress 进行服务路由时，Ingress Controller 基于 Ingress 规则将客户端请求直接转发到 Service 对应的后端 Endpoint（Pod）上，这样会跳过 kube-proxy 设置的路由转发规则，以提高网络转发效率。

通过配置，Ingress 可以为 Service 提供外部可访问的 URL、对其流量作负载均衡、终止 SSL/TLS，以及基于名称的虚拟托管等能力。Ingress 控制器负责完成 Ingress 的工作，具体实现上通常会使用某个负载均衡器，不过也可以配置边缘路由器或其他前端来帮助处理流量。Ingress 不会随意公开端口或协议。将 HTTP 和 HTTPS 以外的服务开放到 Internet 时，通常使用 `Service.TYpe=NodePort` 或 `Service.Type=LoadBalancer` 类型的 Service。

Ingress 的 API 版本经历多次的变化，它们的配置项也不太一样，分别是：

- **extensions/v1beta1**： 1.16版本之前使用；
- **networking.k8s.io/v1beta1**：1.19版本之前使用；
- **networking.k8s.io/v1**：1.19版本之后使用

### 1.1、IngressClass

Ingress 可以由不同的控制器实现，通常使用不同的配置。每个 Ingress 应当指定一个类，也就是一个对 IngressClass 资源的引用。是一个集群资源。

```yaml
apiVersion: networking.k8s.io/v1
kind: IngressClass
metadata:
  annotations:
    ingressclass.kubernetes.io/is-default-class: "true"   # 设置为默认类
  name: kong
spec:
  controller: ingress-controllers.konghq.com/kong.  # 设置对应的控制器，取决于安装的控制器
```

**IngressClass 作用域**

IngressClass 的作用域取决于所使用的 Ingress 控制器，可以使用集群作用域的参数或某个命名空间作用域的参数。

IngressClass 参数的默认作用域是集群范围。如果设置了 `.spec.parameters` 字段且未设置 `.spec.parameters.scope` 字段，或是将 `.spec.parameters.scope` 字段设置为了 `Cluster`，那么该 IngressClass 所引用的既是一个集群作用域的资源。参数 `kind（和 apiGroup一起）` 指向一个集群作用域的 API 类型（可能是一个定制资源（Custom Resource）），而其 `name` 字段则进一步确定该 API 类型的一个具体的、集群作用域的资源。

**示例文件**

> 此 IngressClass 的配置定义了一个名为 "`external-config-1`" 的ClusterIngressParameter（API 组为 `k8s.example.net`）资源中。这项定义告诉 Kubernetes 去寻找一个集群作用域的参数资源。

```yaml
apiVersion: networking.k8s.io/v1
kind: IngressClass
metadata:
  name: external-lb-1
spec:
  controller: example.com/ingress-controller
  parameters:
    scope: Cluster
    apiGroup: k8s.example.net
    kind: ClusterIngressParameter
    name: external-config-1
```





### 1.2、Ingress 版本区别

如果 Kubernetes 版本低于 1.19，可以使用 `networking.k8s.io/v1beta1` 替代，配置可以参考以下配置，只有 backend 配置不一样

#### 1.2.1、v1beta1 示例文件

```yaml
apiVersion: networking.k8s.io/v1beta1    # api 版本号
kind: Ingress                            # 资源类型
metadata:
  annotations:                           # 注解
    kuberentes.io/ingress.class: "nginx"   # 使用哪个 ingress 控制器，集群可以有多个
    nginx.ingress.kubernetes.io/rewrite-target: "/$1"  # nginx 的 rewrite 功能
    nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"  # 后端如果是 https 协议需要配置，默认 http
  name: nginx  # 名称
spec:
  rules:   #一个Ingress可以配置多个rules
  - host: www.zhangzhuo.com  #域名配置，可以不写，匹配*， *.bar.com
    http:
      paths:   #相当于nginx的location配合，同一个host可以配置多个path / /abc
      - backend:    #后端svc定义
          serviceName: nginx-a  #svc名称
          servicePort: 80       #svc端口
        path: "/a/(.*)"         #匹配的路径
      - backend:
          serviceName: nginx-b
          servicePort: 80
        path: "/b/(.*)"
  tls:    #需要https加密
  - hosts:  #主机域名列表，可以配置多个
    - www.zhangzhuo.com
    - www.zhangzhuo1.com
    secretName: tls #secret证书文件，可以写多个
    secretName: tls1
  
```

#### 1.2.2、v1 示例文件

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx
  annotations:
    kubernetes.io/ingress.class: "nginx"  #使用那个ingress控制器，集群有多个可以选择
    nginx.ingress.kubernetes.io/rewrite-target: "/$1"  #nginx的rewrite功能
    nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"   #后端如果是https协议
spec:
  rules:    #一个Ingress可以配置多个rules
  - host: www.zhangzhuo.com   #域名配置，可以不写，匹配*， *.bar.com
    http: 
      paths:  #相当于nginx的location配合，同一个host可以配置多个path / /abc
      - pathType: Prefix
        path: "/"
        backend:  #后端svc定义
          service:  #service的定义
            name: nginx  #svc名称
            port:
              number: 80 #svc端口
  tls:    #需要https加密
  - hosts:  #主机域名列表，可以配置多个
    - www.zhangzhuo.com
    - www.zhangzhuo1.com
    secretName: tls #secret证书文件，可以写多个
    secretName: tls1
```

## 二、Ingress Controller 是什么

Ingress Controller 需要实现基于不同 HTTP URL 向后转发的负载分发规则，并可以灵活设置 7 层负载分发策略。目前 Ingress Controller 已经有许多实现方案，包括 Nginx、HAProxy、Kong、Traefik、Skipper、Istio 等开源软件的实现，以及公有云 GCE、Azure、AWS 等提供 Ingress 应用网关。

在 Kubernetes 中，Ingress Controller 会持续监控 API Server 的 Ingress 接口（即用户定义的到后端服务的转发规则）的变化。当 Ingress 接口后端的服务信息发生变化时，Ingress Controller 会自动更新其转发规则。

Ingress Controller 可以以 DaemonSet 或 Deployment 模式进行部署，通常可以考虑设置 nodeSelector 或亲和性调度策略将其调度到固定的几个 Node 上提供服务。

对于客户端应用如何通过网络访问 Ingress Controller，通过在容器级别设置 hostPort，将 80 或 443 端口号映射到宿主机上，这样客户端应用可以通过 URL 地址 `http://NodeIP:80` 或 `https://NodeIP:443` 访问 Ingress Controller。也可以配置 Pod 使用 hostNetwork 模式直接监听宿主机网卡的 IP 地址和端口号，或者使用 Service 的 NodePort 将端口号映射到宿主机上。

## 三、Ingress 资源

Ingress 需要指定 `apiVersion`、`kind`、`metadata` 和 `spec` 字段。Ingress 对象的命名空间必须是合法的 DNS 子域名名称。Ingress 经常使用注解（Annotations）来配置一些选项，具体取决于 Ingress 控制器，例如 rewrite-targe 注解。不同的 Ingress 控制器支持不同的注解。Ingress 规约提供了配置负载均衡器或者代理服务器所需要的所有信息。最重要的是，其中包含对所有入站请求进行匹配的规则列表。Ingress 资源仅支持用于转发 HTTP/HTTPS 流量的规则。

### 3.1、 Ingress 示例

```bash
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx-example
  rules:
  - http:
      paths:
      - path: /testpath
        pathType: Prefix
        backend:
          service:
            name: nginx
            prot:
              number: 80
```

### 3.2、Ingress 规则

每个 HTTP 规则都包含以下信息：

- **可选的 host**：此规则基于所指定 IP 地址来匹配所有入站 HTTP 流量，如果提供了 host（例如：nginx.test.com），则 roles 适用于所指定的主机；
- **路径列表（例如：/testpath）**：每个路径都有一个由 `service.name` 和 `service.port.name` 或 `service.port.number` 确定的关联后端。主机和路径都必须与入站请求的内容相匹配，负载均衡器才会将流量引导到所引用的 Service；
- **backend（后端）**：是 Service 和端口名称的组合，或者是通过 CRD 方式来实现的自定义资源后端。对于发往 Ingress 的 HTTP/HTTPS 请求，如果与规则中的主机和路径匹配，则会发送到所列出的后端。

> 通常会在 Ingress 控制器中配置 `defaultBackend（默认后端）`，以便为无法与规约中任何路径匹配的所有请求提供服务。

### 3.3、默认后端

没有设置规则的 Ingress 将所有流量发送到同一个默认后端，而在这种情况下 `.spec.defaultBackend` 则是负责处理请求的那个默认后端。`defaultBackend` 通常是 Ingress 控制器的配置选项，而非在 Ingress 资源中设置。如果未设置 `.spec.rules`，则必须设置 `.spec.defaultBackend`。如果未设置 defaultBackend，那么如何处理与所有规则都不匹配的流量将交由 Ingress 控制器决定。如果 Ingress 对象中主机和路径都没有与 HTTP 请求匹配，则流量被路由到默认后端。

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: default
spec:
  defaultBackend:
    service:
      name: default
      port:
        number: 80
```

## 四、Ingress 规则配置

### 4.1、路径类型

Ingress 中的每个路径都需要有对应的路径类型（Path Type）。未明确设置 `pathType` 的路径无法通过合法性检查。当前支持的路径类型有三种：

- `ImplementationSpecific`：对于这种路径类型，匹配方法取决于 IngressClass。具体实现可以将其作为单独的 pathType 处理或者作为与 Prefix 或 Exact 类型相同的处理；
- `Exact`：精确匹配 URL 路径，且缺粉大小写；
- `Prefix`：基于以 `/` 分割的 URL 路径前缀匹配。匹配区分大小写，并且对路径中各个元素逐个执行匹配操作。路径元素指的是由 `/` 分割符分割的路径中的标签列表。如果每个 P 都是请求路径 P 的元素前缀，则请求与路径 P 匹配。

> 说明：如果路径的最后一个元素是请求路径中最后一个元素的子字符串，则不会被视为匹配（例如；/foo/bar 匹配 /foo/bar/baz，但不匹配 /foo/barbaz）。

**示例：**

| 类型   | 路径                          | 请求路径     | 匹配与否？             |
| ------ | ----------------------------- | ------------ | ---------------------- |
| Prefix | /                             | （所有路径） | 是                     |
| Exact  | /foo                          | /foo         | 是                     |
| Exact  | /foo                          | /bar         | 否                     |
| Exact  | /foo                          | /foo/        | 否                     |
| Exact  | /foo/                         | /foo         | 否                     |
| Prefix | /foo                          | /foo，/foo/  | 是                     |
| Prefix | /foo/                         | /foo，foo/   | 是                     |
| Prefix | /aaa/bb                       | /aaa/bbb     | 否                     |
| Prefix | /aaa/bbb                      | /aaa/bbb     | 是                     |
| Prefix | /aaa/bbb/                     | /aaa/bbb     | 是，忽略尾部斜线       |
| Prefix | /aaa/bbb                      | /aaa/bbb/    | 是，匹配尾部斜线       |
| Prefix | /aaa/bbb                      | /aaa/bbb/ccc | 是，匹配子路径         |
| Prefix | /aaa/bbb                      | /aaa/bbbxyz  | 否，字符串前缀不匹配   |
| Prefix | /，/aaa                       | /aaa/ccc     | 是，匹配 /aaa 前缀     |
| Prefix | /，/aaa，/aaa/bbb             | /aaa/bbb     | 是，匹配 /aaa/bbb 前缀 |
| Prefix | /，/aaa，/aaa/bbb             | /ccc         | 是，匹配 / 前缀        |
| Prefix | /aaa                          | /ccc         | 否，使用默认后端       |
| 混合   | /foo（Prefix），/foo（Exact） | /foo         | 是，优选 Exact类型     |

**多重匹配**

在某些情况下，Ingress 中会有多条路径与同一个请求匹配。这时匹配路径最长者优先。如果仍然有两条同等的匹配路径，则精确路径类型优先于前缀路径类型。

### 4.2、主机名通配符

主机名可以是精确匹配（例如："`foo.bar.com`"）或者使用通配符来匹配（例如："`*.foo.com`"）。精确匹配要求 HTTP host 头部字段与 host 字段值完全匹配。通配符匹配则要求 HTTP host 头部字段与通配符规则中的后缀部分相同。

| 主机      | host 头部       | 匹配与否？                          |
| --------- | --------------- | ----------------------------------- |
| *.foo.com | bar.foo.com     | 基于相同的后缀匹配                  |
| *.foo.com | bar.bar.foo.com | 不匹配，通配符仅覆盖了一个 DNS 标签 |
| *.foo.com | foo.com         | 不匹配，通配符仅覆盖了一个 DNS 标签 |

## 五、Ingress 类型

### 5.1、单个 Service 支持

现有的 Kuberentes 概念允许暴露单个 Service，也可以使用 Ingress 并设置无规则的默认后端来完成这类操作：

**示例文件**

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: test-ingress
spec:
  defaultBackend:
    service:
      name: test
      port:
        number: 80
```

> Ingress 控制器和负载均衡器的 IP 地址分配操作可能需要一两分钟。在此之前，通常会看到地址字段的取值为 <pending>

### 5.2、Fanout

一个 Fanout 配置根据请求的 HTTP URL 将来自同一 IP 地址的流量路由到多个 Service。Ingress 允许将负载均衡器的数量降至最低

![image-20240317190518706](https://typora-1306014148.cos.ap-nanjing.myqcloud.com/markdown/image-20240317190518706.png)

**示例文件**

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: simple-fanout-example
spec:
  rules:
  - host: foo.bar.com
    http:
      paths:
      - path: /foo
        pathType: Prefix
        backend:
          service:
            name: service1
            port:
              number: 4200
      - path: /bar
        pathType: Prefix
        backend:
          service:
            name: service2
            port:
              number: 8080
```

此 Ingress 控制器构造一个特定于实现的负载均衡器来供 Ingress 使用，前提是 Service（Service1、Service2）存在。当它完成负载均衡器的创建时，会在 describe 中的 Address 字段看到负载均衡器的地址。

### 5.3、基于名称的虚拟主机

基于名称的虚拟主机支持将针对多个主机名的 HTTP 流量路由到同一 IP 地址上。

![image-20240317190931475](https://typora-1306014148.cos.ap-nanjing.myqcloud.com/markdown/image-20240317190931475.png)

**示例文件**

以下示例中让后台负载均衡器基于 host 头部字段来路由请求

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: name-virtual-host-ingress
spec:
  rules:
  - host: foo.bar.com
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: service1
            port:
              number: 80
  - host: bar.foo.com
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: service2
            port:
              number: 80
```

如果所创建的 Ingress 资源没有在 rules 中定义主机，则规则可以匹配向 Ingress 控制器 IP 地址的所有网络流量，而无需基于名称的虚拟主机

**示例文件**

以下示例中，Ingress 对象会将请求 `test.aaa.com` 的流量路由到 service1，将请求 `test.bbb.com` 的流量路由到 service2 ，而将其他流量路由到 service3

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: test
spec:
  rules:
  - host: test.aaa.com
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: service1
            port:
              number: 80
  - host: test.bbb.com
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: service2
            port:
              number: 80
  - http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: service3
            port:
              number: 80
```

### 5.4、TLS

可以通过设置包含了 TLS 私钥和证书的 Secret 来保护 Ingress。Ingress 资源只支持一个 TLS 端口 443，并假定 TLS 连接终止于 Ingress 节点（与 Service 及其 Pod 间的流量都以明文传输）。如果 Ingress 中的 TLS 配置部分指定了不同主机，那么它们将通过 SNI TLS 扩展指定的主机名（如果 Ingress 支持 SNI）在同一端口上进行复用。TLS Secret 的数据中必须包含键名为 `tls.crt` 的证书和键名为 `tls.key` 的私钥，才能用于 TLS 目的。

**Secret 示例文件**

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: testsecret-tls
  namespace: default
data:
  tls.crt: base64 编码的证书
  tls.key: base64 编码的私钥
type: kubernetes.io/tls
```

在 Ingress 中引用此 Secret 将会告诉 Ingress 控制器使用 TLS 加密从客户端到负载均衡器的通道。要确保所创建的 TLS Secret 创建自包含 `test.tls.com` 的公共名称（Common Name，CN）的证书。公共名称也成为全限定域名（Fully Qualified Damain Name，FQDN）。

> 不能指定默认规则使用 TLS，因为这样做需要为所有可能的子域名签发证书。因此 TLS 字段中的 hosts 的取值需要与 rules 字段中的 host 完全匹配。

**Ingress 示例文件**

```yaml
apiVsersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: tls-example-ingress
  namespace: default
spec:
  tls:
  - hosts:
    - test.tls.com
    secretName: testsecret-tls
  rules:
  - host: test.tls.com
    http:
      paths:
      - path: /
        patyType: Prefix
        backend:
          service:
            name: service1
            port:
              number: 80
```

### 5.5、负载均衡

Ingress 控制器启动引导时使用一些适用于所有 Ingress 的负载均衡策略设置，例如负载均衡算法、后端权重方案等。更高级的负载均衡概念（例如持久会话、动态权重）尚未通过 Ingress 公开。可以通过用于 Service 的负载均衡器来获取这些功能。值的注意的是，尽管健康检查不是通过 Ingress 直接暴露的，在 Kubernetes 中存在就绪态探针这类等价的概念，供实现相同的目的