# Master增加和删除污点

### 背景

使用`kubeadm`部署的`kubernetes`集群，其中`master`节点上默认拒绝`pod`调度运行在此上面。官方术语是：master默认被赋予了一个`taints`(污点)，那么想让master也成为工作节点，有一下两种方法

1. 去掉taints（污点）  备注：生产环境不推荐
2. 让pod能够容忍该节点上的污点



## 去掉污点（taints）

#### 查看节点taints

```
kubectl describe node NODE_NAME | grep Taint
```

![image-20230825165247448](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825165247448.png)



#### 删除节点taints

```
kubectl taint node k8s-master node-role.kubernetes.io/master:NoSchedule-
```

![image-20230825165328300](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825165328300.png)



## 增加污点（taints）

增加污点，禁止pod调度到该节点上

```
kubectl taint node k8s-master node-role.kubernetes.io/master:NoSchedule
```



## pod容忍

需要在pod上声明 `Toleration`。下面两个 `Toleration`都被设置为可以容忍具有污点的Node，使Pod能够调度到有污点的节点上。

```
tolerations:
- key: "key"
  operator: "Equal"
  value: "value"
  effect: "NoSchedule"
```

或者

```
tolerations:
- key: "key"
  operator: "Exists"
  effect: "NoSchedule"
```

`pod`的`Toleration`声明中的key和effect需要与Taint的设置保持一致，并且满足一下条件之一：

- operator 的值是 Exists（无需指定 value）
- operator 的值是 Equal 并且 value相等

比如上面的例子可以这样写：

```
tolerations:
- key: "node-role.kubernetes.io/maste"
  operator: "Equal"
  value: "node-role.kubernetes.io/maste"
  effect: "NoSchedule"
```

或者

```
tolerations:
- key: "node-role.kubernetes.io/master"
  operator: "Exists"
  effect: "NoSchedule"
```

# k8s Label 介绍及增删改查

- 

#### 常用的label实例如下：

- 版本标签：`release: stable` 和 `release: canary`
- 环境标签：`environment: dev`、`environment: qa` 和 `environment: production`
- 架构标签：`tier: frontend`、`tier: backend` 和 `tier: middleware`
- 分区标签：`partition: customerA`和`partition: customerB`
- 质量管控标签：`track: daily` 和 `track: weekly`

### Node上label的增删改查

#### 查看node的标签

```
[root@k8s-master ~]# kubectl get nodes --show-labels
```

![image-20230825165416293](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825165416293.png)



#### 增加node的标签

增加语法：`kubectl label nodes <node-name> <label-name>=<label-value>`

```
[root@k8s-master ~]# kubectl label nodes k8s-node01 zone=north
```

![image-20230825165454011](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825165454011.png)



#### 修改node的标签

在原有增加标签基础上，添加 `--overwrite`参数

修改语法：`kubectl label nodes <node-name> <label-name>=<label-value> --overwrite`

> 注意：修改的时候需要添加`--overwrite`参数，不然会报错

```
[root@k8s-master ~]# kubectl label nodes k8s-node01 zone=nginx --overwrite
```

![image-20230825165548485](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825165548485.png)



#### 删除node的标签

删除label，在命令行最后执行label的key-name并于一个减号相连

删除语法：`kubectl label nodes <node-name> <label-name>-`

```
[root@k8s-master ~]# kubectl label nodes k8s-node01 zone-
```

![image-20230825165641986](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825165641986.png)