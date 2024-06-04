### 一、集群操作

#### 1、查看客户端和服务器版本

如果 kubectl 客户端版本低于 k8s 服务端版本太多，使用 kubectl 管理 k8s 时可能会出现位置异常。如果升级了 k8s 版本最好更新下使用的 kubectl 版本。

```bash
[root@localhost ~]# kubectl version -o=json | jq -r '{client:.clientVersion.gitVersion, server:.serverVersion.gitVersion}'
{
  "client": "v1.25.16",
  "server": "v1.25.16"
}
```

#### 2、查看服务器可用的 api 资源

此命令列出了 kubectl 当前连接的 k8s 集群中所有可用的 api 资源。每个 api 资源代表了集群中的一种对象类型（如 Pod、Service、Deployment 等）。从这里可以查看支持的 api 资源名称、简写、是否时命名空间级别的资源等信息。

> 不同 k8s 集群可能由于集群版本不一样或存在用户自定义的 api 和资源从而会有所差别

如果 api 资源有简写，我们在操作资源时可以使用简写来代替提高效率。例如获取 configmaps 资源时可以用 `kubectl get cm` 来代替。同时由于 NAMESPACE 列显示为 true，我们知道 configmaps 是命名空间级别资源，创建 configmaps 时需要指定命名空间。同一个命名空间中名称唯一仅在此命名空间内可用。

```bash
[root@localhost ~]# kubectl api-resources | head   # 只查询前 10 行，后面还有很多
NAME                              SHORTNAMES         APIVERSION                             NAMESPACED   KIND
bindings                                             v1                                     true         Binding
componentstatuses                 cs                 v1                                     false        ComponentStatus
configmaps                        cm                 v1                                     true         ConfigMap
endpoints                         ep                 v1                                     true         Endpoints
events                            ev                 v1                                     true         Event
limitranges                       limits             v1                                     true         LimitRange
namespaces                        ns                 v1                                     false        Namespace
nodes                             no                 v1                                     false        Node
persistentvolumeclaims            pvc                v1                                     true         PersistentVolumeClaim
```

#### 3、查看资源的帮助文档

通过该命令，可以查看特定资源对象的字段、属性、标签和其他相关信息，帮助我们了解资源对象的结构和可用选项。编写资源清单文件时非常有用。

```bash
[root@localhost ~]# kubectl explain pods
[root@localhost ~]# kubectl explain cm
[root@localhost ~]# kubectl explain pods.spec.volumes.configMap
```

#### 4、切换 kubectl 默认命名空间

```bash
kubectl config set-context --current --namespace=kube-system
```

### 二、集群资源操作

#### 1、查看指定命名空间下所有的 pod 资源

```bash
[root@localhost ~]# kubectl get pods -n kube-system 
NAME                                      READY   STATUS    RESTARTS   AGE
calico-kube-controllers-9559bb7d9-52wvh   1/1     Running   0          18d
calico-node-gtfb5                         1/1     Running   0          18d
calico-node-jf2w9                         1/1     Running   0          18d
calico-node-jsqqk                         1/1     Running   0          18d
calico-node-nktfc                         1/1     Running   0          18d
coredns-5bbbb7dd44-gn9cv                  1/1     Running   0          3d10h
metrics-server-8597f6dd8f-jgf59           1/1     Running   0          18d
metrics-server-8597f6dd8f-zb5fn           1/1     Running   0          18d
```

#### 2、查看拥有指定标签的 Pod 资源

如果 pod 有多个标签可以继续后面增加指定多个 -l 选项（-l k1=v1 -l k2=v2）

```bash
[root@localhost ~]# kubectl get pods -n kube-system -l k8s-app=calico-node
NAME                READY   STATUS    RESTARTS   AGE
calico-node-gtfb5   1/1     Running   0          18d
calico-node-jf2w9   1/1     Running   0          18d
calico-node-jsqqk   1/1     Running   0          18d
calico-node-nktfc   1/1     Running   0          18d

[root@localhost ~]# kubectl get pods -n kube-system -l k8s-app=calico-node -l pod-template-generation=1
NAME                READY   STATUS    RESTARTS   AGE
calico-node-gtfb5   1/1     Running   0          18d
calico-node-jf2w9   1/1     Running   0          18d
calico-node-jsqqk   1/1     Running   0          18d
calico-node-nktfc   1/1     Running   0          18d
```

#### 3、查看指定 Pod 容器信息

查看指定 Pod 内运行的容器名称及其镜像

```bash
[root@localhost ~]# kubectl get pods calico-node-gtfb5 -n kube-system -o jsonpath='{range .spec.containers[*]}{.name}{"\t"}{.image}{"\n"}{end}' |column -t

calico-node  10.202.43.191:5000/kubernetes/calico/node:v3.25.2
```

#### 4、查看指定 Node 上运行的所有 Pod 列表

```bash
# 先查看节点列表拿到节点名称，然后筛选字段
[root@localhost ~]# kubectl get nodes
NAME            STATUS   ROLES    AGE   VERSION
10.202.43.114   Ready    master   18d   v1.25.16
10.202.43.46    Ready    <none>   18d   v1.25.16
10.202.43.55    Ready    <none>   18d   v1.25.16
10.202.43.56    Ready    <none>   18d   v1.25.16

[root@localhost ~]# kubectl get pods -A --field-selector=spec.nodeName=10.202.43.46
NAMESPACE     NAME                                               READY   STATUS             RESTARTS       AGE
argocd        argocd-dex-server-66478ff869-64q2q                 1/1     Running            0              5d5h
argocd        argocd-notifications-controller-7bfcf4d855-9jzvv   1/1     Running            0              5d5h
default       nacos-859fbfd869-bjh7m                             1/1     Running            0              3d23h
kube-system   calico-node-nktfc                                  1/1     Running            0              18d
kube-system   coredns-5bbbb7dd44-gn9cv                           1/1     Running            0              3d10h
kube-system   metrics-server-8597f6dd8f-jgf59                    1/1     Running            0              18d
openebs       openebs-ndm-node-exporter-vnxgd                    1/1     Running            0              16d
software      gogs-5f958bcc55-t2wnw                              1/1     Running            0              3d23h
software      mysql-8d96868cd-xxhqk                              1/1     Running            0              3d23h
software      nginx-6755dcd448-m2lwr                             1/1     Running            0              3h10m
```

#### 5、查看 Pod 的 uid

有时候用需要到节点排查 Pod 异常的时候，kubectl 日志中显示的 uid 而不是 Pod 名称，所以需要进行转换

```bash
[root@localhost ~]# kubectl get pod calico-node-jf2w9 -n kube-system  -o jsonpath='{.metadata.uid}'
bd305a9c-a554-4f33-9be1-5c70fad33a4c

[root@localhost ~]# kubectl get pods -l k8s-app=calico-node -n kube-system -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.metadata.namespace}{"\t"}{.metadata.uid}{"\n"}{end}'
calico-node-gtfb5       kube-system     ba0a6516-d8b9-4ce5-ab7a-387677157a6d
calico-node-jf2w9       kube-system     bd305a9c-a554-4f33-9be1-5c70fad33a4c
calico-node-jsqqk       kube-system     8f2aaa6b-605f-4025-86e3-165e447c413c
calico-node-nktfc       kube-system     8a0b1beb-5e55-4375-8a97-4b15c28f67b3
```

#### 6、查看所有 Pod 使用的镜像

```bash
kubectl get pods --all-namespace -o jsonpath='{range .items[*]}{"\n"}{.metadata.name}{":\t"}{range .spec.containers[*]}{.image}{", "}{end}{end}' |sort
```

#### 7、设置 Node 禁止调度

```bash
kubectl taint nodes xx.xx.xx.xx key=value:NoSchedule
```

#### 8、驱逐 Node 节点上的 Pod

```bash
kubectl drain xxxxx --delete-local-data --force --ignore-daemonsets
```

#### 9、指定调度 Pod 到固定主机

```bash
...
      nodeSelector:
        kubernetes.io/hostname: 11.53.101.105
...
```

### 三、问题排查

#### 1、 查看 Pod 中指定容器的日志

查看 Pod 中有哪些容器

```bash
[root@localhost ~]# kubectl get pods postgresql-6f747cd99c-qxs5w -n software -o jsonpath='{range .spec.containers[*]}{.name}{"\t"}{.image}{"\n"}{end}' |column -t
postgresql  10.202.43.191:5000/soft/postgres/postgres:12.18
```

如果不指定容器名则默认查看 Pod 中第一个容器的日志。-c 选项指定要查看的容器名称，--tail 选项指定要输出最后多少行日志（默认从头打印所有日志），-f 选项指定保持一直监听日志输出

```bash
[root@localhost ~]# kubectl logs postgresql-6f747cd99c-qxs5w -n software -c postgresql --tail=5 -f
2024-04-23 13:24:04.403 UTC [72596] LOG:  could not receive data from client: Connection reset by peer
2024-04-23 13:37:10.821 UTC [73377] LOG:  could not receive data from client: Connection reset by peer
2024-04-23 13:37:10.821 UTC [73378] LOG:  could not receive data from client: Connection reset by peer
2024-04-23 13:41:32.966 UTC [58409] LOG:  could not receive data from client: Connection reset by peer
2024-04-23 13:46:27.878 UTC [73870] LOG:  could not receive data from client: Connection reset by peer
```

#### 2、在容器中执行命令

```bash
[root@localhost ~]# kubectl exec postgresql-6f747cd99c-qxs5w -n software -c postgresql -- date
Tue Apr 23 01:51:04 PM UTC 2024

[root@localhost ~]# kubectl exec postgresql-6f747cd99c-qxs5w -n software -c postgresql -- /bin/sh -c 'echo "hello" >/tmp/test.txt'
[root@localhost ~]# kubectl exec postgresql-6f747cd99c-qxs5w -n software -c postgresql -- cat /tmp/test.txt 
hello
```

#### 3、进入容器中交互执行命令

```bash
[root@localhost ~]# kubectl exec -it postgresql-6f747cd99c-qxs5w -n software -c postgresql -- bash
root@postgresql-6f747cd99c-qxs5w:/# date
Tue Apr 23 01:52:43 PM UTC 2024
```

#### 4、复制文件到容器内

```bash
[root@localhost ~]# kubectl cp anaconda-ks.cfg postgresql-6f747cd99c-qxs5w:/tmp/anaconda-ks.cfg -n software -c postgresql
```

#### 5、复制容器内的文件到本地

```bash
[root@localhost ~]# kubectl cp postgresql-6f747cd99c-qxs5w:/tmp/anaconda-ks.cfg /tmp/anaconda-ks.cfg -n software -c postgresql
tar: Removing leading `/' from member names    # 忽略此错误即可
```

### 四、部署相关

#### 1、基于清单文件创建及删除资源

```bash
# 第一种创建方式
[root@localhost nginx]# kubectl create -f nginx.yaml 
deployment.apps/nginx created
service/nginx created

# 删除
[root@localhost nginx]# kubectl delete -f nginx.yaml 
deployment.apps "nginx" deleted
service "nginx" deleted
```

#### 2、更新 Pod 模版中的镜像

```bash
[root@localhost nginx]# kubectl set image deploy nginx nginx=10.202.43.191:5000/kubernetes/nginx/nginx:1.24.0
deployment.apps/nginx image updated
```

#### 3、更新 Pod 副本数

```bash
[root@localhost nginx]# kubectl scale deploy nginx --replicas=2
deployment.apps/nginx scaled
```

#### 4、重启部署资源

重启指定的资源中的所有 Pod 实例

```bash
[root@localhost nginx]# kubectl rollout restart deploy nginx
deployment.apps/nginx restarted
```

#### 5、对比本地清单文件应用后和线上资源的差异

如果将本地清单文件 apply 后，可以使用 diff 对比和当前线上资源的差异

```bash
[root@localhost nginx]# kubectl apply -f nginx.yaml 
deployment.apps/nginx created
service/nginx created

[root@localhost nginx]# kubectl diff -f nginx.yaml 
diff -u -N /tmp/LIVE-147993013/apps.v1.Deployment.default.nginx /tmp/MERGED-3034723863/apps.v1.Deployment.default.nginx
--- /tmp/LIVE-147993013/apps.v1.Deployment.default.nginx        2024-04-24 11:07:34.258386200 +0800
+++ /tmp/MERGED-3034723863/apps.v1.Deployment.default.nginx     2024-04-24 11:07:34.259386200 +0800
@@ -6,14 +6,14 @@
     kubectl.kubernetes.io/last-applied-configuration: |
       {"apiVersion":"apps/v1","kind":"Deployment","metadata":{"annotations":{},"name":"nginx","namespace":"default"},"spec":{"replicas":1,"selector":{"matchLabels":{"app":"nginx"}},"template":{"metadata":{"labels":{"app":"nginx","run":"nginx"}},"spec":{"containers":[{"image":"docker.gsrecv.com/kubernetes/nginx/nginx:1.24.0","name":"nginx","ports":[{"containerPort":80}]}]}}}}
   creationTimestamp: "2024-04-23T14:04:58Z"
-  generation: 5
+  generation: 6
   name: nginx
   namespace: default
   resourceVersion: "3958341"
   uid: f533d6e9-d4fe-473d-a247-ad68b4315f0e
 spec:
   progressDeadlineSeconds: 600
-  replicas: 2
+  replicas: 1
   revisionHistoryLimit: 10
   selector:
     matchLabels:
@@ -33,7 +33,7 @@
         run: nginx
     spec:
       containers:
-      - image: 10.202.43.191:5000/kubernetes/nginx/nginx:1.24.0
+      - image: docker.gsrecv.com/kubernetes/nginx/nginx:1.24.0
         imagePullPolicy: IfNotPresent
         name: nginx
         ports:
```

### 五、标签和污点

#### 1、查看标签

```bash
[root@localhost nginx]# kubectl get node --show-labels
NAME            STATUS   ROLES    AGE   VERSION    LABELS
10.202.43.114   Ready    master   18d   v1.25.16   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=10.202.43.114,kubernetes.io/os=linux,kubernetes.io/role=master
10.202.43.46    Ready    <none>   18d   v1.25.16   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=10.202.43.46,kubernetes.io/os=linux,software=sofrware
10.202.43.55    Ready    <none>   18d   v1.25.16   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=10.202.43.55,kubernetes.io/os=linux,software=sofrware
10.202.43.56    Ready    <none>   18d   v1.25.16   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=10.202.43.56,kubernetes.io/os=linux,software=sofrware
```

#### 2、给节点添加及删除自定义标签

给 10.202.43.46 节点添加名为 address=beijing 的标签

```bash
[root@localhost nginx]# kubectl label node 10.202.43.46 address=beijing
node/10.202.43.46 labeled
```

删除标签（标签名后面跟 - 号）

```bash
[root@localhost nginx]# kubectl label node 10.202.43.46 address-
node/10.202.43.46 unlabeled
```

#### 3、查看污点

```bash
[root@localhost nginx]# kubectl describe nodes 10.202.43.114 | grep Taints
Taints:             <none>
```

#### 4、添加及删除污点

给节点添加名为 address=beijing:NoSchedule 的污点（不能容忍该污点的 Pod 将不会调度到该节点）

```bash
[root@localhost nginx]# kubectl taint nodes 10.202.43.114 address=beijing:NoSchedule
node/10.202.43.114 tainted
```

删除污点

```bash
[root@localhost nginx]# kubectl taint nodes 10.202.43.114 address-
node/10.202.43.114 untainted
```

#### 5、容忍污点

在 containers 中设置 Pod 容忍度

```bash
spec:
  containers:
  ...
  tolerations:
  - key: "key"
    operator: "Equal"
    value: "value"
    effect: "NoSchedule"
```



