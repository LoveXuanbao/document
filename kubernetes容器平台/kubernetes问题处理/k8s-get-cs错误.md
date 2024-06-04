## kubeadm安装集群kubectl get cs组件状态异常

#### 背景

通过 `kubeadm`安装得kubenets集群，一台`master`，两台`nodes`。

`kubectl get nodes`查看到所有节点状态都是正常的。

![image-20230825164102784](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825164102784.png)

`kubectl get pod -A`,查看所有pod信息，也都是正常。

![image-20230825164202989](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825164202989.png)

#### 问题

`kubectl get cs`查看kubenertes集群组件得状态，发现`controller-manager`和`scheduler`状态为 `Unhealthy`。

![image-20230825164247876](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825164247876.png)



### 排查过程

执行 `netstat -tnlp`查看错误信息中得10252和10251端口是不存在的。

![image-20230825164339983](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825164339983.png)



查看contriller-manager和scheduler配置文件是否禁用非安全端口。文件路径在`/etc/kubernetes/manifests`

![image-20230825164423472](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825164423472.png)



注释掉以下配置，两个文件都是同样的

![image-20230825164514556](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825164514556.png)

![image-20230825164605275](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825164605275.png)



重启 kubelet

```
systemctl restart kubelet
```



`netstat -ntlp`重新查看端口，发现端口已经启动

![image-20230825164701203](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825164701203.png)



`kubectl get cs`重新看组件状态，已经恢复正常

![image-20230825164743178](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825164743178.png)