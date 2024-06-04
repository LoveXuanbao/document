```bash
controller.go:136] failed to ensure node lease exists, will retry in 7s, error: leases.coordination.k8s.io "10.202.43.114" is forbidden: User "system:node:k8s-node-1" cannot get resource "leases" in API group "coordination.k8s.io" in the namespace "kube-node-lease": can only access node lease with the same name as the requesting node
```

## 原因

其他节点复制过来的配置文件包含证书/var/lib/kubelet/pki导致重复了，删除后，重启kubelet服务即可