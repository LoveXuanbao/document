gitlab 下载地址

`https://github.com/kubernetes-sigs/kind/releases`

- 选择不同的版本进行下载，每个版本都支持不同的操作系统



![image-20231023174032086](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20231023174032086.png)



- 注意：不同的 kind 版本安装的 k8s 版本不一致，请参考每个版本中对应的 k8s 版本信息，入下图所示

![image-20231023174057163](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20231023174057163.png)





```
# Kind 创建K8s集群 配置文件
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
name: k8s  #集群命名
featureGates:   #启用kubernetes的功能
  "APIListChunking": true
networking:  #网络配置
  podSubnet: "10.244.0.0/16"  #pod网络配置
  serviceSubnet: "10.96.0.0/12" #svc网络配置
  kubeProxyMode: "ipvs"  #kube-proxy的工作模式
  apiServerAddress: "127.0.0.1"  #API服务器监听地址
  apiServerPort: 6443   #API服务器监听端口
  disableDefaultCNI: true #禁用默认的CNI插件，禁用后不安装需要手动安装
containerdConfigPatches: #私人仓库配置
- |-
  [plugins."io.containerd.grpc.v1.cri".registry.mirrors."192.168.10.10:5000"]
    endpoint = ["http://192.168.10.10:5000"]
nodes:  #节点配置
- role: control-plane
  image: kindest/node:v1.26.0  #选用的镜像可选择不同版本
  extraMounts:  #节点额外的挂载目录用于持久化数据
  - hostPath: /data1/etcd
    containerPath: /var/lib/etcd
  - hostPath: /data1
    containerPath: /data1
  extraPortMappings:  #额外的端口映射
  - containerPort: 80
    hostPort: 80  
    listenAddress: "127.0.0.1"
    protocol: TCP
- role: worker
  image: kindest/node:v1.26.0 
- role: worker
  image: kindest/node:v1.26.0
- role: worker
  image: kindest/node:v1.26.0
```

