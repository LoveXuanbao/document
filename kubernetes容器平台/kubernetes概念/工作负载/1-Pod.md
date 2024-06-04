## 一、什么是Pod

官方文档：https://kubernetes.io/zh/docs/concepts/workloads/pods/

**Pod** 是 Kubernetes 的**最小的业务单元**， 可以简单的理解为是一组、一个或者多个容器沟通，每个 Pod 还包含一个 **Pause容器**；

**Pause容器** 是 Pod 的父容器，它主要负责僵尸进程的回收管理，同时通过 Pause容器 可以使用一个 Pod 里面的不同容器共享存储、网络、PID、IPC 等，容器之间可以使用 **localhost:port** 相互访问，可以使用 volume 等实现数据共享。

Pod 的共享上下文包括一组 Linux 名字空间、控制组（cgroup）和可能一些其他的隔离方面，即用来隔离容器技术。在 Pod 的上下文中，每个独立的应用可能会进一步实施隔离。

根据 Docker 的构造，Pod 可以被建模为一组具有共享**命令空间、卷、IP地址和Port端口**的容器

Kubernetes 集群中的 Pod 主要有两种用法：

- **运行单个容器的 Pod**：每个 Pod 一个容器是最常见的 Kubernetes 用例；在这种情况下，可以将 Pod 看作单个容器的包装器，并且 Kubernetes 直接管理 Pod ，而不是容器；
- **运行多个系统工作的容器的 Pod**：Pod 可以封装有紧密耦合且需要共享资源的**多个并置容器**组成的应用，这些位于同一位置的容器构成一个内聚单元。将多个并置、同管的容器组织到一个 Pod 中是一种相对高级的使用场景，只有在一些场景中，容器之间紧密关联时才应该使用这种模式；

**Pod 通常不是直接创建的，而是使用工作负载资源创建的**

用于管理 Pod 的工作负载

通常不需要直接创建 Pod，甚至单实例 Pod；相反，应该使用诸如 Deployment 或 Job 这类工作负载资源来创建 Pod，如果 Pod 需要跟踪状态，可以考虑 StatefulSet 资源。

每个 Pod 都旨在运行给定应用程序的单个实例。如果希望横向扩展应用程序（例如：运行多个实例以提供更多的资源），则应该使用多个 Pod，每个实例使用一个 Pod；在 Kubernetes 中，这通常被称为**副本（Replication）**；通常使用一种工作负载资源及其控制器来创建和管理一组 Pod 副本

## 二、为什么要引入Pod

- 强依赖的服务需要部署在一起；
- 多个服务需要协同工作
- 兼容其他CRI标准的运行时；

## 三、创建一个Pod

#### 3.1、使用 --dry-run命令生成一个yaml文件

```bash
[root@k8s-master-1 k8s]# kubectl run nginx --image=repository.test.com:8444/kubernetes/nginx/nginx:1.7.9 -oyaml --dry-run > nginx.yaml


# 返回结果
W0525 23:58:04.241448  497595 helpers.go:663] --dry-run is deprecated and can be replaced with --dry-run=client.
```

- 查看生成的文件

```yaml
[root@k8s-master-1 k8s]# cat nginx.yaml 
apiVersion: v1  
kind: Pod  
metadata:  
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: repository.test.com:8444/kubernetes/nginx/nginx:1.7.9
    name: nginx
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

#### 3.2、定义一个nginx.yaml

```yaml
apiVersion: v1  # 必选，API 的版本号
kind: Pod  # 必选，资源的类型 Pod、Deployment、StatefulSet、Service等等
metadata:  # 必选，元数据的信息
  labels:  # 可选，定义标签信息
    run: nginx  # key: value 方式定义
  name: nginx  # 必选，符合 RFC 1035 规范的 Pod 名称
spec:  # 必选，用于定义 pod 的详细信息
  containers:  # 必选，容器列表
  - name: nginx  # 必选，符合 RFC 1035 规范的 Pod 名称
    image: repository.test.com:8444/kubernetes/nginx/nginx:1.7.9  # 必选，指定容器所用的镜像地址
    ports:   # 可选，容器需要暴露的端口号列表
    - containerPort: 80   # 端口号
```

#### 3.3、创建Pod

```bash
[root@k8s-master-1 k8s]# kubectl create -f nginx.yaml 
pod/nginx created
```

#### 3.4、查看Pod状态

```bash
[root@k8s-master-1 k8s]# kubectl get pod 
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          3s
```

#### 3.5、删除pod

```bash
[root@k8s-master-1 k8s]# kubectl delete pod nginx
pod "nginx" deleted
```

#### 3.6、使用 kubectl run 创建一个pod

```bash
[root@k8s-master-1 k8s]# kubectl run nginx --image=repository.test.com:8444/kubernetes/nginx/nginx:1.7.9
pod/nginx created

[root@k8s-master-1 k8s]# kubectl get pod -owide
NAME    READY   STATUS    RESTARTS   AGE   IP              NODE           NOMINATED NODE   READINESS GATES
nginx   1/1     Running   0          79s   172.16.110.11   10.202.43.46   <none>           <none>
```

#### 3.7、访问nginx

```bash
[root@k8s-master-1 k8s]# curl 172.16.110.11
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

## 四、静态 Pod

**静态 Pod （Static Pod）**直接由特定节点上的 kubelet 守护进程管理，不需要 API 服务器看到它们。尽管大多数 Pod 都是通过控制面（例如：Deployment）来管理的，对于静态 Pod 而言，kubelet 直接监控每个 Pod，并在其失效时重启。

静态 Pod 通常绑定到某个节点上的 kubelet，其主要用途时运行自托管的控制面；在自托管场景中，使用 kubelet 来管理各个独立的控制面组件；

kubelet 自动尝试为每个静态 Pod 在 Kubernetes API 服务器上创建一个镜像 Pod。这意味着在节点上运行的 Pod 在 API 服务器上是可见的，但不可以通过 API 服务器来控制；

> 静态 Pod 的 Spec 不能引用其他的 API 对象（例如：ServiceAccount、ConfigMap、Secret等）
>
> 要确保 kubelet 在 API 服务上有创建镜像 Pod 的权限，如果没有，创建请求会被 API 服务拒绝

在部署 Kubernetes 时 kubelet 配置文件中有一个配置用于指定静态 Pod 文件存放位置，存放在这里的 Pod 配置文件会被 kubelet 进行管理。kubelet 会定期的扫描这个文件夹下 YAML/JSON 文件来创建/删除静态 Pod。

- 在 Kubernetes 的配置文件 **kubelet.config.k8s.io/v1beta1** 配置为：**staticPodPath: <目录>**
- 在 kubelet 启动时以参数指定：**--pod-manifest-path**；

在 Pod 启动后使用 **kubectl get pod** 命令可以查看到 Pod，但是并不能进行修改配置，如果需要修改 Pod ，需要到固定的 kubelet 节点修改 Pod 静态文件；

## 五、Pod镜像拉取策略

可以通过 **spec.containers[].imagePullPolicy** 参数指定镜像的拉取策略，目前支持的策略如下：

| 操作方式     | 说明                                                         |
| ------------ | ------------------------------------------------------------ |
| Always       | 总是拉取，当镜像 tag 为 latest 时，且 imagePullPolicy 未配置，默认为 Always |
| Never        | 不管是否存在不会拉取                                         |
| IfNotPresent | 镜像不存在时拉取镜像，如果 tag 为 **非latest**，且 imagePullPolicy 未配置时，默认为 IfNotPresent |

更改镜像的拉取策略

```bash
apiVersion: v1 # 必选，API 的版本号
kind: Pod # 必选，类型 Pod
metadata: # 必选，元数据
  name: nginx # 必选，符合 RFC 1035 规范的 Pod 名称
spec: # 必选，用于定义 Pod 的详细信息
  containers: # 必选，容器列表
    - name: nginx # 必选，符合 RFC 1035 规范的容器名称
      image: repository.test.com:8444/kubernetes/nginx/nginx:1.7.9 # 必选，容器所用的镜像的地址
      imagePullPolicy: IfNotPresent # 可选，镜像拉取策略
      ports: # 可选，容器需要暴露的端口号列表
        - containerPort: 80 # 端口号
```

## 六、Pod重启策略

可以通过 **spec.restartPolicy** 指定容器的重启策略

| 操作方式  | 说明                                      |
| --------- | ----------------------------------------- |
| Always    | 默认策略，容器失效时，自动重启该容器      |
| OnFailure | 容器以不为 0 的状态码终止，自动重启该容器 |
| Never     | 无论何种状态，都不会重启                  |

指定重启策略为 **Never**

```bash
apiVersion: v1 # 必选，API 的版本号
kind: Pod # 必选，类型 Pod
metadata: # 必选，元数据
  name: nginx # 必选，符合 RFC 1035 规范的 Pod 名称
spec: # 必选，用于定义 Pod 的详细信息
  containers: # 必选，容器列表
    - name: nginx # 必选，符合 RFC 1035 规范的容器名称
      image: repository.test.com:8444/kubernetes/nginx/nginx:1.7.9 # 必选，容器所用的镜像的地址
      imagePullPolicy: IfNotPresent # 可选，镜像拉取策略
      command:   # 可选，容器启动执行的命令，覆盖docker镜像的 entrypoint
        - sleep
        - "3"
      ports: # 可选，容器需要暴露的端口号列表
        - containerPort: 80 # 端口号
  restartPolicy: Never
```

## 七、Pod的三种探针

| 种类           | 说明                                                         |
| -------------- | ------------------------------------------------------------ |
| startupProbe   | Kubernetes 1.16 新加的探测方式，用于判断容器内的应用程序是否已经启动，如果配置了 startupProbe，就会先禁用其他探测，直到它成功为止。如果探测失败，kubelet 会杀死容器，之后根据重启策略进行处理；如果探测成功或没有配置 startupPrpbe，则状态为成功，之后就不再探测 |
| livenessProbe  | 用于探测容器是否在运行，如果探测失败，kubelet 会 "杀死" 容器并根据重启策略进行相应的处理。如果未指定该探针，将默认为 Success |
| readinessProbe | 一般用于探测容器内的程序是否健康，即判断容器是否为就绪（Ready）状态。如果是，则可以处理请求，反之 Endpoints Controller 将从所有的 Service 的 Endpoints 中删除此容器所在的 Pod 的 IP 地址。如果未指定，将默认为 Success |

### 7.1、Pod探针的实现方式

| 实现方式        | 说明                                                         |
| --------------- | ------------------------------------------------------------ |
| ExecAction      | 在容器内执行一个指定的命令，如果命令返回值为 0 ，则认为容器健康 |
| TCPSocketAction | 通过 TCP 连接检查容器指定的端口，如果端口开放，则认为容器健康 |
| HTTPGetAction   | 对指定的 URL 进行 Get 请求，如果状态码在 200～400 之间，则认为容器健康 |

### 7.2、readlinessProbe探针

创建一个没有探针的pod

```bash
apiVersion: v1 # 必选，API 的版本号
kind: Pod # 必选，类型 Pod
metadata: # 必选，元数据
  name: nginx # 必选，符合 RFC 1035 规范的 Pod 名称
spec: # 必选，用于定义 Pod 的详细信息
  containers: # 必选，容器列表
    - name: nginx # 必选，符合 RFC 1035 规范的容器名称
      image: repository.test.com:8444/kubernetes/nginx/nginx:1.7.9 # 必选，容器所用的镜像的地址
      imagePullPolicy: IfNotPresent # 可选，镜像拉取策略
      command:   # 可选，容器启动执行的命令，覆盖docker镜像的 entrypoint
        - sh
        - -c
        - sleep 10; nginx g "daemon off;"
      ports: # 可选，容器需要暴露的端口号列表
        - containerPort: 80 # 端口号
  restartPolicy: Never
```

配置健康检查

- 下面例子中配置了一个就绪探针，kubelet 会在容器启动 10 秒后发送第一个就绪探针，探针会尝试连接 nginx 容器的 80 端口。如果探测成功，这个 Pod 会被标记为就绪状态， kubelet 将继续每隔 5 秒进行一次探测

```bash
apiVersion: v1 # 必选，API 的版本号
kind: Pod # 必选，类型 Pod
metadata: # 必选，元数据
  name: nginx # 必选，符合 RFC 1035 规范的 Pod 名称
spec: # 必选，用于定义 Pod 的详细信息
  containers: # 必选，容器列表
    - name: nginx # 必选，符合 RFC 1035 规范的容器名称
      image: repository.test.com:8444/kubernetes/nginx/nginx:1.7.9 # 必选，容器所用的镜像的地址
      imagePullPolicy: IfNotPresent # 可选，镜像拉取策略
      command:   # 可选，容器启动执行的命令，覆盖docker镜像的 entrypoint
        - sh
        - -c
        - sleep 10; nginx -g "daemon off;"
      readinessProbe:  # 可选，健康检查，注意：三种健康检查方式同时只能使用一种
        tcpSocket:  # 接口检测方式
          port: 80  # 检查端口
        initialDelaySeconds: 10  # 初始化时间，健康检查延迟执行时间
        timeoutSeconds: 2   # 超时时间
        periodSeconds: 5  # 检测间隔
        successThreshold: 1  # 检测成功为1次表示就绪
        failureThreshold: 2  # 检测失败为2次表示未就绪
      ports: # 可选，容器需要暴露的端口号列表
        - containerPort: 80 # 端口号
  restartPolicy: Never
```

### 7.3、livenessProbe探针

- 下面例子中配置了一个就绪探针和存活探针，kubelet 会在容器启动 10 秒后发送第一个就绪探针，探针会尝试连接 `nginx` 容器的 80 端口。如果探测成功，这个 Pod 会被标记为就绪状态， kubelet 将继续每隔 5 秒进行一次探测。kubelet 会在容器启动 15 秒之后，进行存活探测，与就绪探针类似，存活探针会尝试连接 nginx 的 80 端口，如果存活探测失败，容器会被重新启动。

```bash
apiVersion: v1 # 必选，API 的版本号
kind: Pod # 必选，类型 Pod
metadata: # 必选，元数据
  name: nginx # 必选，符合 RFC 1035 规范的 Pod 名称
spec: # 必选，用于定义 Pod 的详细信息
  containers: # 必选，容器列表
    - name: nginx # 必选，符合 RFC 1035 规范的容器名称
      image: repository.test.com:8444/kubernetes/nginx/nginx:1.7.9 # 必选，容器所用的镜像的地址
      imagePullPolicy: IfNotPresent # 可选，镜像拉取策略
      command:   # 可选，容器启动执行的命令，覆盖docker镜像的 entrypoint
        - sh
        - -c
        - sleep 10; nginx -g "daemon off;"
      ports: # 可选，容器需要暴露的端口号列表
        - containerPort: 80 # 端口号
      readinessProbe:  # 可选，健康检查，注意：三种健康检查方式同时只能使用一种
        httpGet:  # 接口检测方式
          path: /index.html  # 检查路径
          port: 80  # 检查端口
          scheme: HTTP  # HTTP 或者 HTTPS
          # httpHeaders: # 可选，检查的请求头
          # - name: end-user
          #   value: Json
        initialDelaySeconds: 10  # 初始化时间，健康检查延迟执行时间
        timeoutSeconds: 2   # 超时时间
        periodSeconds: 5  # 检测间隔
        successThreshold: 1  # 检测成功为1次表示就绪
        failureThreshold: 2  # 检测失败为2次表示未就绪
      livenessProbe:
        httpGet:
          path: /index.html
          port: 80
          scheme: HTTP
        initialDelaySeconds: 15
        timeoutSeconds: 2
        periodSeconds: 5
        successThreshold: 1
        failureThreshold: 2
  restartPolicy: Always
```

### 7.4、startupProbe探针

启动探针一般常用于启动时需要较长的初始化时间的容器

```bash
apiVersion: v1 # 必选，API 的版本号
kind: Pod # 必选，类型 Pod
metadata: # 必选，元数据
  name: nginx # 必选，符合 RFC 1035 规范的 Pod 名称
spec: # 必选，用于定义 Pod 的详细信息
  containers: # 必选，容器列表
    - name: nginx # 必选，符合 RFC 1035 规范的容器名称
      image: repository.test.com:8444/kubernetes/nginx/nginx:1.7.9 # 必选，容器所用的镜像的地址
      imagePullPolicy: IfNotPresent # 可选，镜像拉取策略
      command:   # 可选，容器启动执行的命令，覆盖docker镜像的 entrypoint
        - sh
        - -c
        - sleep 30; nginx -g "daemon off;"
      startupProbe:
        tcpSocket:
           port: 80
        initialDelaySeconds: 10  # 初始化时间，健康检查延迟执行时间
        timeoutSeconds: 2   # 超时时间
        periodSeconds: 5  # 检测间隔
        successThreshold: 1  # 检测成功为1次表示就绪
        failureThreshold: 20  # 检测失败为2次表示未就绪
      readinessProbe:  # 可选，健康检查
        httpGet:  # 接口检测方式
          path: /index.html  # 检查路径
          port: 80  # 检查端口
          scheme: HTTP  # HTTP 或者 HTTPS
          # httpHeaders: # 可选，检查的请求头
          # - name: end-user
          #   value: Json
        initialDelaySeconds: 2  # 初始化时间，健康检查延迟执行时间
        timeoutSeconds: 2   # 超时时间
        periodSeconds: 5  # 检测间隔
        successThreshold: 1  # 检测成功为1次表示就绪
        failureThreshold: 2  # 检测失败为2次表示未就绪
      livenessProbe:
        httpGet:
          path: /index.html
          port: 80
          scheme: HTTP
        initialDelaySeconds: 2
        timeoutSeconds: 2
        periodSeconds: 5
        successThreshold: 1
        failureThreshold: 2
  restartPolicy: Always
```

## 八、preStop和postStart

**Prestop** 和 **postStart** 定义了为容器的生命周期时间挂接处理函数，当一个容器启动后， Kubernetes 将立即发送 **postStart** 事件；在容器被终结之前，kubernetes 将发送一个 **preStop** 事件。容器可以为每个事件指定一个处理程序。

- 在下面配置文件中，可以看到 postStart 命令在容器 /usr/shar 目录下写入文件 message；preStop 命令负责优雅的终止 nginx 服务。当因为失效而导致终止容器时，这个处理方式很有用。

```bash
apiVersion: v1 # 必选，API 的版本号
kind: Pod # 必选，类型 Pod
metadata: # 必选，元数据
  name: nginx # 必选，符合 RFC 1035 规范的 Pod 名称
spec: # 必选，用于定义 Pod 的详细信息
  containers: # 必选，容器列表
    - name: nginx # 必选，符合 RFC 1035 规范的容器名称
      image: repository.test.com:8444/kubernetes/nginx/nginx:1.7.9 # 必选，容器所用的镜像的地址
      imagePullPolicy: IfNotPresent # 可选，镜像拉取策略
      command:   # 可选，容器启动执行的命令，覆盖docker镜像的 entrypoint
        - sh
        - -c
        - sleep 30; nginx -g "daemon off;"
      lifecycle:
        postStart:
          exec:
            command:
              - sh
              - -c
              - echo Hello from the postStart handler > /usr/share/message
        preStop:
          exec:
            command:
              - sh
              - -c
              - nginx -s quit; while killall -0 nginx; do sleep 1; done
```

创建 pod 

```bash
[root@k8s-master-1 study]# kubectl create -f nginx.yaml
pod/nginx created
```

查看pod中的容器是否已经运行

```bash
[root@k8s-master-1 study]# kubectl get pod
NAME                                     READY   STATUS    RESTARTS   AGE
nginx                                    1/1     Running   0          2s
```

使用 bash 命令连接到 pod 里的容器

```bash
[root@k8s-master-1 study]# kubectl exec -it nginx -- bash
root@nginx:/#
```

查看 postStart 处理函数创建的 message 文件

```bash
root@nginx:/# cat /usr/share/message
Hello from the postStart handler
```

### 说明

Kubernetes 在容器创建后立即发送 postStart 事件，然而，postStart 处理函数的调用不保证早于容器的入口点（entrypoint）的执行。postStart 处理函数于容器的代码时异步执行的，但 Kubernetes 的容器管理逻辑会一直阻塞等待 postStart 处理函数执行完毕。只有 postStart 处理函数执行完毕，容器的状态才会变成 Running

Kubernetes 在容器结束前立即发送 preStop 事件，除非 Pod 宽限期限超时，Kubernetes 的容器管理逻辑会一直阻塞等待 preStop 处理函数执行完毕。

Kubernetes 只有在一个 Pod 或该 Pod 中的容器结束（Terminated）的时候才会发送 preStop 事件，这意味着在 Pod 完成（Completed）时 preStop 的事件处理逻辑不会被触发。

## 九、Pod生命周期

Pod 遵循预定义的声明周期，起始于 Pending 阶段，如果至少其中有一个主要容器正常启动，则进入 Running，之后取决于 Pod 中是否有容器以失败状态结束而进入 Succeeded 或者 Failed 阶段。

在 Pod 运行期间，kubelet 能够重启容器以处理一些失效场景。在 Pod 内部，Kubernetes 跟踪不同容器的状态并确定使 Pod 重新变得健康所需要采取得动作。

Pod 在其生命周期中只会被调度一次，一旦 Pod 被调度（分派）到某个节点， Pod 会一直在该系欸按运行，直到 Pod 停止或者被终止。

### 9.1、Pod 生命期

和一个个独立得应用容器一样，Pod也被认为是相对临时性（而不是长期存在）的实体。Pod 会被创建、赋予一个唯一的 ID（UID），并被调度到节点，并在终止（根据重启策略）或删除之前一直运行在该节点。

如果一个节点死掉了，调度到该节点的 Pod 也被计划在给定超时期限结束后删除；

Pod 自身不具有自愈能力，如果 Pod 被调度到某节点而该节点之后失效，Pod 会被删除；类似的，Pod无法在因节点资源耗尽或者节点维护而被驱逐期间继续存活。Kubernetes 使用一种高级抽象来管理这些相对而言可随时丢弃的 Pod 实例，称作控制器。

任何给定的 Pod（由 UID 定义）从不会被”重新调度（rescheduled）“到不同的节点；相反，这一 Pod 可以被一个新的、几乎完全相同的 Pod 替换掉。如果需要，新 Pod 的名字可以不变，但是其 UID 不不同。

如果某物声称其生命期与某 Pod 相同，例如存储卷，这就意味着该对象在此 Pod（UID 亦相同）存在期间也一直存在。如果 Pod 因为任何原因被删除，甚至某完全相同的替代 Pod 被创建时，这个相关的对象（例如这里的卷）也会被删除并重建。

### 9.2、Pod 的阶段

Pod 的 status 字段是一个 PodStatus 对象，其中包含一个 phase 字段。

Pod 的阶段（Phase）是 Pod 在其生命周期中所处位置的简单宏观概述。该阶段并不是对容器或 Pod 状态的综合汇总，也不是为了成为完成的状态机。

phase 字段表示 Pod 所处的节点的值如下：

| 取值                                             | 描述                                                         | 排查方法                                                     |
| ------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **Pending**（悬决）                              | Pod 已被 Kubernetes 系统接收，但有一个或者多个容器尚未创建亦未运行。此阶段包括等待 Pod 被调度的时间和通过网络下载镜像的时间 | 可以使用 **kubectl describe pod** 查看 Pod 的 Events 验证 Pod 为什么长时间处于这个状态 |
| **Running**（运行中）                            | Pod 已经绑定到了某个节点，Pod 中所有的容器都已被创建。至少有一个容器仍在运行，或者正处于启动或重启转台。 | 表示 Pod 已经正常启动，可以使用 **kubectl logs pod** 查看 Pod 的日志 |
| **Succeeded**（成功）                            | Pod 中的所有容器已成功终止，并且不会再次重启                 | 表示 Pod 已经正常启动，可以使用 **kubectl logs pod** 查看 Pod 的日志 |
| **Failed**（失败）                               | Pod 中的所有都已终止，并且至少有一个容器时以失败终止，也就是说，容器以非 0 状态退出或者被系统终止。 | 可以通过 **kubectl logs pod** 和 **kubectl describe pod** 查看 Pod 日志和状态 |
| **Unknows**（未知）                              | 因为某些原因无法取得 Pod 的状态。这种情况通常是因为与 Pod 所在的主机通信失败 | 可以通过 **kubectl get pod -owide** 找到运行这个 Pod 的节点排查节点运行是否正常，主要时是网络是否正常 |
| **Terminating**（已终止）                        | 当一个Pod 被删除时，会展示这个 Pod 的状态，Pod 被赋予一个可以体面终止的期限，默认为 30 秒， | 可以使用 --force 参数来强制终止 Pod。**kubectl delete pod xxx --force** |
| **ImagePullBackOffErrImagePull**（镜像拉取错误） | 镜像拉取失败，一般是由于镜像不存在、网络不同或者需要登录认证引起的。 | 可以使用 **kubectl describe pod** 查看 Pod 的 Events 验证具体的错误原因 |
| **CrashLoopBackOff**（循环崩溃）                 | 容器启动失败，一般为部署服务的原因，代码问题、启动命令不正确、健康检查不通过等等 | 可以使用 **kubectl logs pod** 查看 Pod 的日志查看具体原因    |
| **ContainerCreating**（创建容器）                | Pod 正在创建，一般是正在拉取镜像                             | 如果长时间处于该阶段，可以通过 **kubectl describe pod** 查看 Pod 的 Events 状态 |
| **Completed**（完成）                            | 容器内部主进程退出，一般计划任务执行结束会显示该状态         | 可以使用 **kubectl logs pod** 查看 Pod 的日志                |
| **SysctlForbidden**                              | Pod 自定义了内核配置，但是 kubelet 没有添加内核配置或配置的内核参数不支持， | 如果长时间处于该阶段，可以通过 **kubectl describe pod** 查看 Pod 的 Events 状态 |
| **OOMKilled**                                    | 容器内存溢出，一般是容器的内存 **Limit** 设置的过小，或者程序本身有内存溢出 | 可以使用 **kubectl logs pod** 查看 Pod 的日志                |

> 如果某节点死掉或者与集群中其他节点失联，Kubernetes 会实施一种策略，将市区的节点上运行的所有的 Pod 的 phase 设置为 Failed。

### 9.3、Pod 的状态

在 Pod 运行期间，kubelet 能够重启容器以处理一些失效场景。在 Pod 内部，Kubernetes 跟踪不同容器的状态并确定使用 Pod 重新变得健康所需要采取的动作。

Kubernetes 会跟踪 Pod 中每个容器的状态，就像它跟踪 Pod 总体上的阶段一样，你可以使用容器生命周期回调来在容器生命周期中的特定时间点触发事件。

一旦调度器将 Pod 分派到某个节点，kubelet 就通过容器运行时开始为 Pod 创建容器。容器的状态有三种：**Waiting（等待）、Running（运行中）和 Terminated（已终止）**

**每个状态都有特定的含义**

- **Waiting（等待）**：如果容器并不处在 Running 或 Terminated 状态之一，它就处在 Waiting 状态。处于 Waiting 状态的容器仍在运行它完成启动所需要的操作，例如：从某个容器镜像仓库拉取容器镜像，或者向容器应用 Secret 数据等等。当使用 kubectl 来查询包含 Waiting 状态的容器的 Pod 时，也会看到一个 Reason 字段，其中给出了容器处于等待状态的原因。
- **Running（运行中）**：Running 状态表明容器正在执行状态并且没有问题发生。如果配置了 postStart 回调，那么该回调已经执行且已完成。如果使用 kubectl 来查询包含 Running 状态的容器的 Pod 时，也会看到关于容器进入 Running 状态的信息。
- **Terminated（已终止）**：处于 Terminated 状态的容器已经开始执行或者正常结束或者因为某些原因失败。如果你使用 kubectl 来查询包含 Terminated 状态的容器的 Pod 时，就胡i看到容器进入此状态的原因、退出代码以及容器执行期间的起止事件。如果容器配置了 preStop 回调，则该回调会在容器进入 Terminated 状态之间执行。

### 9.4、Pod 的状况

Pod 有一个 PodStatus 对象，其中包含一个 PodConditions 数组，Pod 可能通过也可能未通过其中的一些状况测试。kubelet 管理如下：

- **PodScheduled**：Pod 已经被调度到某节点；
- **PodReadyToStartContainers**：Pod 沙箱被成功创建并且配置了网络（Beta 特性，默认启用）；
- **ContainersReady**：Pod 中所有容器都已就绪；
- **Initialized**：所有的 Init 容器都已成功完成；
- **Ready**：Pod 可以为请求提供服务，并且应该被添加到对应服务的负载均衡池中；

| 字段名称           | 描述                                                         |
| ------------------ | ------------------------------------------------------------ |
| type               | Pod 状况的名称                                               |
| lastProbeTime      | 上次探测 Pod 状况时的时间戳                                  |
| lastTransitionTime | Pod 上次从一种状态转换到另一种状态时的时间戳                 |
| reason             | 机器可读的、驼峰编码（UpperCamelCase）的文字，表述上次状况变化的原因 |
| message            | 人类可读的消息，给出上次状态转换的详细信息                   |

### 9.5、Pod 的终止

由于 Pod 所代表的是在集群中节点上运行的进程，当不再需要这些进程时允许其体面地终止是很重要的。一般不应武断的使用 **kill** 信号终止它们，导致这些进程没有机会完成清理操作。

设计的目标是令你能够请求删除进程，并且知道进程何时被终止，同时也能够确保删除操作终将完成。当你请求删除某个 Pod 时，集群会记录并跟踪 Pod 的体面终止周期，而不是直接强制地杀死 Pod。在存在强制关闭设施的前提下，kubelet 会尝试体面地终止 Pod。

通常情况下，Pod 体面终止的过程为：kubelet 先发送一个带有体面超时期限的 TERM（又名 SIGTERM）信号到每个容器中的主进程，将请求发送到容器运行时来尝试停止 Pod 中的容器。停止容器的这些请求容器运行时以异步方式处理。这些请求的处理顺序无法被保证。许多容器运行时遵循容器镜像内定义的 **STOPSIGNAL** 的值，如果不同，则发送容器镜像中配置的 **STOPSIGNAL**，而不是 **TERM** 信号。一旦超出了体面终止限期，容器运行时会向所有剩余进程发送 **KILL** 信号，之后 Pod 就会被从 API 服务器上移除。如果 **kubelet** 或者容器运行时的管理服务在等待进程终止期间被重启，集群会从头开始重试，赋予 Pod 完成的体面终止限期。

**强制终止 Pod**

默认情况下，所有的删除操作都会附有 30 秒宽限期限。**kubectl delete** 命令支持 **--grace-period=<seconds>** 选项，允许重载默认值，设定自己希望的期限值。将宽限期限强制设置为 0 意味着立即从 API 服务器删除 Pod。如果 Pod 仍然运行于某节点上，强制删除操作会触发 kubelet 立即执行清理操作。

必须在设置 **grace-period=0** 的同时额外设置 **--force** 参数才能发起强制删除请求。

```bash
kubectl delete pod POD_NAME--grace-period=0 --force
```

执行强制删除操作时，API 服务器不在等待来自 kubelet 的关于 Pod 已经在原来运行的节点上终止执行的确认消息。API 服务器直接删除 Pod 对象，这样新的与之同名的 Pod 即可被创建。在节点侧，被设置为立即终止的 Pod 仍然会在被强行杀死之前获得一点点的宽限时间。

**注意：**马上删除时不等待确认正在运行的资源已被终止。这些资源可能会无限期地继续在集群上运行。

## 十、零宕机发布 Pod

如果需要实现服务的零宕机发布，在 Pod 设置中需要做的步骤如下：

1. 首先配置服务的健康检测探针，三种类型的探针需要进行配置，尤其是服务的启动探针。探针最好是使用 API 接口进行探测，其他方式不推荐；
2. 并且进行测试确定服务的启动后就绪所需要的时间，如果服务本身没有问题在就绪探针探测成功后就表示服务正常启动成功；
3. 把服务正常启动的时间配置为删除 Pod 时的容忍时间，可以写的比启动时间长一些；
4. 之后配置容器回调，主要是 **preStop** 参数，大多数微服务都会使用注册中心，使用脚本的方式把自己在注册中心取消注册，以防删除的 Pod 接收新的请求，如果是使用 Service 资源进行服务发现的这一步可以不用做。最后执行 **sleep** 命令暂停这个容器，让容器尽可能的处理完之前的连接请求。sleep 的时间可以根据自己的情况设置时长，但是 Pod 的最大容忍时间是 **terminationGracePeriodSeconds** 来设置的。

当然这里的零宕机设置只是 Pod 中所需的必要设置，在实际环境中并不会直接使用 Pod，而是使用其他资源控制器来管理 Pod。一些零宕机的其他功能都有控制器来实现。

## 十一、Init 容器

每个 Pod 都可以包含多个容器，应用运行在这些容器里面，同时 Pod 也可以有一个或多个先于应用容器启动的 Init 容器。

Init 容器与普通的容器非常像，除了如下两点：

1. 它们总是运行到完成；
2. 每个都必须在下一个启动之前成功完成；

如果 Pod 的 Init 容器失败，kubelet 会不断的重启该 Init 容器直到该容器成功为止。然而，如果 Pod 对应的 **restartPolicy** 值为 "Never"，并且 Pod 的 Init 容器失败，则 Kubernetes 会将整个 Pod 状态设置为失败。

如果为一个 Pod 指定了多个 Init 容器，这些容器会按照顺序逐个运行。每个 Init 容器必须运行成功，下一个才能够运行。当所有的 Init 容器运行完成时，Kubernetes 才会为 Pod 初始化应用容器并像平常一样运行。

在所有的 Init 容器没有成功之前，Pod 将不会变成 **Ready** 状态。Init 容器的端口将不会在 Service 中进行聚焦。正在初始化中的 Pod 处于 **Pending** 状态，但会将状况 **Initializing** 设置为 false。

## 十二、Pod 示例文件

```yaml
apiVersion: v1  # 必选，API的版本号
kind: Pod       # 必选，类型Pod
metadata:       # 必选，元数据
  name: pod     # 必选，符合RFC 1035规范的Pod名称
  namespace: default # 可选，Pod所在的命名空间，不指定默认为default，可以使用-n 指定namespace 
  labels:       # 可选，标签选择器，一般用于过滤和区分Pod
    app: pod    #可以写多个
  annotations:  #可选，注释列表，可以写多个
    app: nginx
spec:           # 必选，用于定义容器的详细信息
  initContainers:  #初始化容器，在容器启动之前执行的一些初始化操作
  - name: init-echo
    command:
    - sh
    - -c
    - echo "I am InitContainer for init some configuration"
    image: 192.168.10.254:5000/init/alpine:latest
    imagePullPolicy: IfNotPresent
  containers:    # 必选，容器列表
  - name: nginx  # 必选，符合RFC 1035规范的容器名称
    image: 192.168.10.254:5000/init/nginx:latest    # 必选，容器所用的镜像的地址
    imagePullPolicy: IfNotPresent     # 可选，镜像拉取策略, IfNotPresent: 如果宿主机有这个镜像，那就不需要拉取了. Always: 总是拉取, Never: 不管是否存储都不拉去，默认为Always
    workingDir: /usr/share/nginx/html       # 可选，容器的工作目录
    ports:  # 可选，容器需要暴露的端口号列表
    - name: http    # 端口名称
      containerPort: 80     # 端口号
      protocol: TCP # 端口协议，默认TCP
    env:    #可选，环境变量配置列表
    - name: TZ      # 变量名
      value: Asia/Shanghai # 变量的值
    resources:      # 可选，资源限制和资源请求限制
      limits:       # 最大限制设置
        cpu: 200m  #1核CPU=1000m
        memory: 100Mi
      requests:     # 启动所需的资源
        cpu: 200m
        memory: 100Mi
    startupProbe: # 探测容器是否启动
      exec:        # 执行容器命令检测方式
        command: 
        - ls
        - /tmp/zhangzhuo/1/
      initialDelaySeconds: 30   # 初始化时间
      timeoutSeconds: 1     # 超时时间
      periodSeconds: 3      # 检测间隔
      successThreshold: 1 # 检查成功为1次表示就绪
      failureThreshold: 3 # 检测失败3次表示未就绪
    readinessProbe: # 可选，就绪探针。
      httpGet:      # httpGet检测方式
        path: / # 检查路径
        port: 80        # 监控端口
      initialDelaySeconds: 3
      timeoutSeconds: 1 
      periodSeconds: 3  
      successThreshold: 1 
      failureThreshold: 3 
    livenessProbe:  # 可选，存活探针
      tcpSocket:    # 端口检测方式
        port: 80
      initialDelaySeconds: 60       
      timeoutSeconds: 2     
      periodSeconds: 5      
      successThreshold: 1 
      failureThreshold: 3 
    lifecycle:    #回调配置
      postStart:  #容器启动后执行  
        exec:     #执行命令
          command:
          - sh
          - -c
          - 'mkdir /tmp/zhangzhuo/1 -p'
      preStop:    #容器终止前执行
        httpGet:  #执行http请求    
              path: /
              port: 80
  restartPolicy: Always   # 可选，默认为Always，容器故障或者没有启动成功，那就自动重启该容器，Onfailure: 容器以不为0的状态终止，自动重启该容器, Never:无论何种状态，都不会重启，默认为Always
  nodeSelector:  #可选，指定Node节点
    type: normal #填写node标签
  terminationGracePeriodSeconds: 30 #Pod删除停止时最大容忍时间，默认30s
  dnsPolicy: ClusterFirstWithHostNet  #Pod的dns策略,Default,None,ClusterFirst,ClusterFirstWithHostNet
  hostNetwork: true  #使用宿主机网络，默认不使用
  hostIPC: true  #使用宿主机IPC空间，默认不使用
  hostPID: true  #使用宿主机PID空间，默认不使用
```

