# Argo-cd介绍

## 1. Argo-是什么

- 用于kubernetes运行工作流、管理集群和正确执行GitOps的开源工具

## 2. Argo核心功能

- 将应用程序自动部署到指定的目标环境
- 能够管理和部署到多个集群
- 用于授权的多租户和RBAC策略
- 回滚到Git仓库中提交的任何应用程序配置
- 应用资源健康状态分析



## 部署

参考 github 中的官方说明

```bash
https://github.com/argoproj/argo-cd/releases
```





### 获取默认密码

### 第一种方式

#### 1. 查看argocd的pod服务

```bash
kubectl get pod -n argocd | grep argocd-server

argocd-server-55984f776d-m22sv                      1/1     Running   0          22m
```

#### 2. 进入argocd-server容器中

```bash
kubectl exec -it argocd-server-55984f776d-m22sv -n argocd -- bash
```

#### 3. 执行以下命令查看默认密码

```bash
argocd@argocd-server-55984f776d-m22sv:~$ argocd admin initial-password
Xx0HoG3jSzwisTXW
```

#### 4. 修改默认密码



### 第二种方式

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
```

### 注册一个集群来部署应用程序

#### 首先列出当前kubeconfig中的所有集群上下文

```bash
kubectl config get-contexts -o name

# 返回结果
kubernetes-admin@kubernetes
```





```bash
argocd repo add http://10.202.43.110:10888/gogs/argocd.git --username <username> --password <password>
```





## 安装gogs

### 1. docker启动gogs服务

```bash
docker run -d -it --name gogs \
    -p 10022:22 \
    -p 10888:3000 \
    -v /var/gogs:/data/gogs \
    repository.test.com:8444/kubernetes/gogs/gogs:0.14.0
```

### 2.浏览器访问配置

第一次访问的时候，都要先配置一遍，生产环境建议选择专门的数据库存储信息，测试环境可以选择sqlite数据库

![image-20230825165742604](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825165742604.png)

![](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825165822500.png)

![image-20230825165954041](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825165954041.png)

然后点击立即安装，进入到gogs

### 创建gogs仓库

```bash
mkdir /data/argocd
cd /data/argocd
touch README.md
git init
git add README.md
git commit -m "first commit"
git remote add origin http://xx.xx.xx.xx:10888/gogs/argocd.git
git push -u origin master

# 返回结果
Counting objects: 3, done.
Writing objects: 100% (3/3), 201 bytes | 0 bytes/s, done.
Total 3 (delta 0), reused 0 (delta 0)
Username for 'http://xx.xx.xx.xx:10888': gogs   # 输入用户名
Password for 'http://gogs@xx.xx.xx.xx:10888':   # 输入密码
To http://10.202.43.110:10888/gogs/argocd.git
 * [new branch]      master -> master
Branch master set up to track remote branch master from origin.
```





```yaml
apiVersion: v1
kind: Secret
metadata:
  name: private-repo
  namespace: argocd
  labels:
    argocd.argoproj.io/secret-type: repository
stringData:
  type: git
  url: http://10.202.43.110:10888/gogs/argocd
  password: gogs
  username: 1qaz2wsx
```







```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: demo
  namespace: argocd
spec:
  project: default
  source:
    repoURL: http://10.202.43.110:10888/gogs/argocd.git
    targetRevision: master
    path: default
  destination:
    server: https://kubernetes.default.svc
    namespace: default
```

