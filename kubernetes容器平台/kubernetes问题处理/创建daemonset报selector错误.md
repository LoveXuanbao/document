## k8s 创建 daemonset资源报错 

## 错误信息

```
[root@k8s-master work]# kubectl create -f daemonset.yaml
error: error validating "daemonset.yaml": error validating data: ValidationError(DaemonSet.spec): missing required field "selector" in io.k8s.api.apps.v1.DaemonSetSpec; if you choose to ignore these errors, turn validation off with --validate=false
```

![image-20230825163650845](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825163650845.png)



### 解决办法

在 `spec`中，添加 selector.matchLables

- 原文件

![image-20230825163742222](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825163742222.png)

- 修改后

![image-20230825163822553](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825163822553.png)



最后，重新创建即可

![image-20230825163908430](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825163908430.png)