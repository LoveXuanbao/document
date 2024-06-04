

prometheus 监控页面  Targets显示 DOWN，错误提示如下

```bash
server returned HTTP status 403 Forbidden
```

#### 解决办法：

在kube-controller-manager 的启动参数添加下面两个参数：

```bash
      --authentication-kubeconfig=/etc/kubernetes/kube-controller-manager.kubeconfig \
      --authorization-kubeconfig=/etc/kubernetes/kube-controller-manager.kubeconfig \
```

然后执行重启即可

```bash
systemctl daemon-reload && systemctl restart kube-controller-manager
```



