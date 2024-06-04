# 查看所有pod使用得镜像

```
kubectl get pods --all-namespace -o jsonpath='{range .items[*]}{"\n"}{.metadata.name}{":\t"}{range .spec.containers[*]}{.image}{", "}{end}{end}' |sort
```



# 指定调度pod到固定主机

```
...
      nodeSelector:
        kubernetes.io/hostname: 11.53.101.105
...
```



# 查看 node 节点

```
kubectl get nodes
```

# 驱逐 node 节点上的 pod

```
kubectl drain xxxxx --delete-local-data --force --ignore-daemonsets
```

# 删除 node

```
 kubectl delete nodes xxxxx
```

# 设置node禁止调度

```
kubectl taint nodes 11.10.32.148 key=value:NoSchedule
```

# 资源限制Cgroup目录

```
mkdir -p /sys/fs/cgroup/cpu,cpuacct/kubelet.slice
mkdir -p /sys/fs/cgroup/memory/kubelet.slice
mkdir -p /sys/fs/cgroup/systemd/kubelet.slice
mkdir -p /sys/fs/cgroup/pids/kubelet.slice
mkdir -p /sys/fs/cgroup/cpu,cpuacct/kubelet.slice
mkdir -p /sys/fs/cgroup/cpuset/kubelet.slice

// 系统预留
mkdir -p /sys/fs/cgroup/cpu,cpuacct/system-reserved
mkdir -p /sys/fs/cgroup/memory/system-reserved
mkdir -p /sys/fs/cgroup/systemd/system-reserved
mkdir -p /sys/fs/cgroup/pids/system-reserved
mkdir -p /sys/fs/cgroup/cpu,cpuacct/system-reserved
mkdir -p /sys/fs/cgroup/cpuset/system-reserved
mkdir -p /sys/fs/cgroup/hugetlb/system-reserved

// kubelet预留
mkdir -p /sys/fs/cgroup/cpu,cpuacct/kube-reserved
mkdir -p /sys/fs/cgroup/memory/kube-reserved
mkdir -p /sys/fs/cgroup/systemd/kube-reserved
mkdir -p /sys/fs/cgroup/pids/kube-reserved
mkdir -p /sys/fs/cgroup/cpu,cpuacct/kube-reserved
mkdir -p /sys/fs/cgroup/cpuset/kube-reserved
mkdir -p /sys/fs/cgroup/hugetlb/kube-reserved
```

# k8s 创建镜像拉取任务secrets

```
`for i in `kubectl get secrets --all-namespaces|grep myregistrykey|awk '{print $1}'`; do kubectl delete secret myregistrykey -n $i; kubectl create secret docker-registry myregistrykey --docker-server=192.168.124.43:8002 --docker-username=admin --docker-password=admin123 -n $i ; done
```

or

```
#!/bin/bash
#

for i in `kubectl get secrets --all-namespaces|grep myregistrykey|awk '{print $1}'`; do
    kubectl delete secret myregistrykey -n $i
    kubectl create secret docker-registry myregistrykey --docker-server=192.168.124.43:8002 --docker-username=admin --docker-password=admin123 -n $i
done
```

# 创建PV和PVC

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: aniu-test
spec:

  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  storageClassName: test-test
  nfs:
    path: /data/share/aniu
    server: 10.202.61.75

---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: aniu-test-pv
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: test-test
```

# 创建镜像拉取secrets

```
`for i in `kubectl get secrets --all-namespaces|grep myregistrykey|awk '{print $1}'`; do kubectl delete secret myregistrykey -n $i; kubectl create secret docker-registry myregistrykey --docker-server=192.168.124.43:8002 --docker-username=admin --docker-password=admin123 -n $i ; done
```

or

```
#!/bin/bash
#

for i in `kubectl get secrets --all-namespaces|grep myregistrykey|awk '{print $1}'`; do
    kubectl delete secret myregistrykey -n $i
    kubectl create secret docker-registry myregistrykey --docker-server=192.168.124.43:8002 --docker-username=admin --docker-password=admin123 -n $i
done
```

# POD反亲和性

```yaml
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            - labelSelector:
                matchExpressions:
                  - key: name
                    operator: In
                    values:
                      - sso-cas-svc
              topologyKey: kubernetes.io/hostname
```





