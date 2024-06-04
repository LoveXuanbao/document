## 一、扩缩容工作负载

官方网址：https://kubernetes.io/zh-cn/docs/concepts/workloads/autoscaling/

在 Kubernetes 中，可以根据当前的资源需求扩缩工作负载，这可以让 Kubernetes 集群更灵活、更高效地面对资源需求的变化。

当扩缩工作负载时，可以增加或减少工作负载所管理的副本数量，或者就地调整副本的可用资源。

第一种手段称为**水平扩缩**，第二种称为**垂直扩缩**；扩缩工作负载有手动和自动两种方式，取决于具体的使用情况。

## 二、自动扩缩工作负载

在 Kubernetes 中自动扩缩的概念是指自动更新管理一组 Pod 的能力（例如：Deployment）

Horizontal Pod Autoscaler（HPA）是一个 Pod 水平自动扩缩，可以基于 CPU 利用率自动扩缩 ReplicationController、Deployment、ReplicaSet 和 StatefulSet 中的 Pod 数量，除了 CPU 利用率，也可以基于其他应用程序提供的自定义度量指标来执行自动扩缩。Pod 自动扩缩不适用于无法扩缩的对象，比如：DaemonSet。

<img src="https://typora-1306014148.cos.ap-nanjing.myqcloud.com/markdown/image-20240314212931251.png" alt="image-20240314212931251" style="zoom: 50%;" />

## 三、HPA 接口类型

HPA 的 API 版本有三个，分别是：

- HPA v1 为稳定版自动水平伸缩，只支持 CPU 指标；
- v2 为 beta 版本，分为 v2beta1（支持 CPU、内存和自定义指标）；
- v2beta2 （支持 CPU、内存、自定义指标 Custom 和额外指标 ExternalMetrics）；



## 四、v1 版本扩缩容测试

>  前置条件
>
> - 只能对副本控制器使用（Deployment、ReplicaSet、ReplicationController、StatefulSet）；
> - 需要安装`metrics-server` 采集 Pod 监控指标，集群可以使用 `kubectl top` 命令；
> - Pod 资源定义重必须启动资源限制 `resources.limits、resources.requests`；
> - 控制器选择器中必须匹配到的 Pod 只能一个，不能是多个，部分会导致 HPA 控制器获取不到监控指标，HPA 无法工作；

### 4.1、Deployment 和 Service 示例文件

```YAML
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
  name: nginx-deploy
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - image: repository.test.com:8444/kubernetes/nginx/nginx:1.7.9
        name: nginx
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: 10m

---
apiVersion: v1
kind: Service
metadata:
  name: nginx-deploy
  labels:
    app: nginx
spec:
  ports:
  - port: 80
  selector:
    app: nginx
```

### 4.2、HPA 示例文件

```yaml
apiVersion: autoscaling/v1      # 必须，API 版本
kind: HorizontalPodAutoscaler   # 必须，资源类型
metadata:                       # 必须，元数据定义
  name: nginx-deploy            # 必须，HPA 名字
  namespace: default            # 必须，命名空间
spec:                           # 实际定义的资源内容
  maxReplicas: 10               # 最大 Pod 数
  minReplicas: 1                # 最小 Pod 数
  scaleTargetRef:               # 选择匹配的控制器设置
    apiVersion: apps/v1         # 控制器的 API 版本
    kind: Deployment            # 控制器的资源类型
    name: nginx-deploy          # 控制器名称
  targetCPUUtilizationPercentage: 50   # 扩缩容条件，CPU 负载超过的值的百分比
```

### 4.3、HPA 创建命令

一般 HPA 的 v1 版本创建的参数比较少，可以直接用命令创建

```bash
kubectl autoscale 控制器类型 控制器名称 --min=Pod最小数 --max=Pod最大数 --cpu-percent=CPU负载阈值百分比

# 示例
kubectl autoscale deploy nginx-deploy --min=1 --max=10 --cpu-percent=10


```

### 4.4、创建 HPA 及 Deployment

```bash
[root@k8s-master-1 test]# kubectl create -f nginx-deploy.yaml 
deployment.apps/nginx-deploy created
service/nginx-deploy created
[root@k8s-master-1 test]# kubectl autoscale deploy nginx-deploy --min=1 --max=10 --cpu-percent=10
horizontalpodautoscaler.autoscaling/nginx-deploy autoscaled
[root@k8s-master-1 test]# 
[root@k8s-master-1 test]# kubectl get pod 
NAME                           READY   STATUS    RESTARTS   AGE
nginx-deploy-c4df7c787-z9vq6   1/1     Running   0          35s
[root@k8s-master-1 test]# kubectl get hpa
# # 有时候在没有获取到指标之前，TARGETS 会显示 unknown，稍等片刻，等 metrics-server 获取到指标即可，如果负载一直为空，请检查控制器是否匹配了多个Pod
NAME           REFERENCE                 TARGETS         MINPODS   MAXPODS   REPLICAS   AGE
HPA名称         选择的控制器                当前CPU负载      最小Pod数  最大Pod数  当前Pod数   运行时间
nginx-deploy   Deployment/nginx-deploy   <unknown>/10%   1         10        0          11s
[root@k8s-master-1 test]# kubectl get hpa
NAME           REFERENCE                 TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
nginx-deploy   Deployment/nginx-deploy   0%/10%    1         10        1          17s

[root@k8s-master-1 test]# kubectl get svc
NAME           TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
kubernetes     ClusterIP   172.10.0.1      <none>        443/TCP   7d3h
nginx-deploy   ClusterIP   172.10.25.171   <none>        80/TCP    4m37s
```

### 4.5、模拟高并发请发

```bash
# 循环访问，模拟大量请求进来，IP地址为 kubectl get svc 看到的 nignx-deploy的IP
while true; do curl http://172.10.174.107 > /dev/null; done
```

### 4.6、查看是否自动扩容

```bash
# 持续观察发现 CPU 使用率已经增高，Pod 副本数也开始增加
[root@k8s-master-1 test]# kubectl get hpa -w
NAME           REFERENCE                 TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
nginx-deploy   Deployment/nginx-deploy   0%/10%    1         10        1          6m42s
nginx-deploy   Deployment/nginx-deploy   320%/10%   1         10        1          6m52s
nginx-deploy   Deployment/nginx-deploy   330%/10%   1         10        4          7m7s
nginx-deploy   Deployment/nginx-deploy   97%/10%    1         10        8          7m22s

# 查看 Pod，也可以看到副本数增加
[root@k8s-master-1 test]# kubectl get pod 
NAME                           READY   STATUS    RESTARTS   AGE
nginx-deploy-c4df7c787-6qgd9   1/1     Running   0          29s
nginx-deploy-c4df7c787-6z4ns   1/1     Running   0          29s
nginx-deploy-c4df7c787-89l4d   1/1     Running   0          44s
nginx-deploy-c4df7c787-9m64s   1/1     Running   0          29s
nginx-deploy-c4df7c787-ds9gz   1/1     Running   0          44s
nginx-deploy-c4df7c787-jdnc8   1/1     Running   0          44s
nginx-deploy-c4df7c787-ltgzn   1/1     Running   0          14s
nginx-deploy-c4df7c787-t4lmn   1/1     Running   0          14s
nginx-deploy-c4df7c787-vd78v   1/1     Running   0          29s
nginx-deploy-c4df7c787-z9vq6   1/1     Running   0          8m4s
```

### 4.7、停止请求

```bash
# 停止请求，会发现 CPU 负载已经降低，在经过一点时间后，HPA 会自动缩容 Pod，到最小 Pod 数
[root@k8s-master-1 test]# kubectl get hpa -w
NAME           REFERENCE                 TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
nginx-deploy   Deployment/nginx-deploy   0%/10%    1         10        10         8m55s
nginx-deploy   Deployment/nginx-deploy   0%/10%    1         10        1          12m
[root@k8s-master-1 test]# kubectl get pod 
NAME                           READY   STATUS    RESTARTS   AGE
nginx-deploy-c4df7c787-z9vq6   1/1     Running   0          16m
```

## 五、注意事项

有些场景是不使用 CPU 扩缩容副本的；在大多数微服务架构中，都是前端接收请求，之后交予后端进行处理后写入数据库。如果数据库的负载过高导致后端服务的 CPU 负载过高。我们在后端使用了 HPA 这会导致后端增加副本数，但是根本原因并不是后端服务的问题，增加副本数反而会导致数据库负载继续增加，有可能导致数据库最后宕机。所以 HPA 的 v1 版本并不可以随意使用，要根据实际情况来进行使用。
