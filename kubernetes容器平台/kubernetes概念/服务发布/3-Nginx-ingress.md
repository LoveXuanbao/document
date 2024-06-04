## 一、Ingress-Nginx

Ingress-nginx 是 Kubernetes 官方维护的 Ingress 项目，当然，Nginx 官方也维护了一个自己的 Ingres ，名称为 nginx-ingress。

Ingress-nginx 官方文档：https://kubernetes.github.io/ingress-nginx/deploy/#bare-metal-clusters



## 二、安装 Ingress-nginx

**在线安装**

> 注意：在线安装集群一定是要能能够联网的，而且国内不一定能够正常下载镜像，最好选择离线安装

```bash
kubectl apply -f  https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.10.0/deploy/static/provider/baremetal/deploy.yaml
```

**离线安装**

```bash
# 首先先下载 yaml 文件
wget https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.10.0/deploy/static/provider/baremetal/deploy.yaml

# 过滤出来镜像，
[root@k8s-master-1 ~]#grep -r "image:" deploy.yaml 
        image: registry.k8s.io/ingress-nginx/controller:v1.10.0@sha256:42b3f0e5d0846876b1791cd3afeb5f1cbbe4259d6f35651dcc1b5c980925379c
        image: registry.k8s.io/ingress-nginx/kube-webhook-certgen:v1.4.0@sha256:44d1d0e9f19c63f58b380c5fddaca7cf22c7cee564adeff365225a5df5ef3334
        image: registry.k8s.io/ingress-nginx/kube-webhook-certgen:v1.4.0@sha256:44d1d0e9f19c63f58b380c5fddaca7cf22c7cee564adeff365225a5df5ef3334

# 官方镜像可能国内拉去不到，可以使用代理拉取。
docker pull k8s.dockerproxy.com/ingress-nginx/controller:v1.10.0@sha256:42b3f0e5d0846876b1791cd3afeb5f1cbbe4259d6f35651dcc1b5c980925379c
docker pull k8s.dockerproxy.com/ingress-nginx/kube-webhook-certgen:v1.4.0@sha256:44d1d0e9f19c63f58b380c5fddaca7cf22c7cee564adeff365225a5df5ef3334

#，拉取后修改成自己的镜像仓库地址
docker tag k8s.dockerproxy.com/ingress-nginx/controller:v1.10.0@sha256:42b3f0e5d0846876b1791cd3afeb5f1cbbe4259d6f35651dcc1b5c980925379c repository.test.com:8444/kubernetes/ingress/ingress-nginx/controller:v1.10.0
docker tag k8s.dockerproxy.com/ingress-nginx/kube-webhook-certgen:v1.4.0@sha256:44d1d0e9f19c63f58b380c5fddaca7cf22c7cee564adeff365225a5df5ef3334 repository.test.com:8444/kubernetes/ingress/ingress-nginx/kube-webhook-certgen:v1.4.0

# 上传至自己的镜像仓库
docker push repository.test.com:8444/kubernetes/ingress/ingress-nginx/controller:v1.10.0
docker push repository.test.com:8444/kubernetes/ingress/ingress-nginx/kube-webhook-certgen:v1.4.0

#  修改 yaml 文件
sed -i 's#registry.k8s.io/ingress-nginx/controller:v1.10.0@sha256:42b3f0e5d0846876b1791cd3afeb5f1cbbe4259d6f35651dcc1b5c980925379c#repository.test.com:8444/kubernetes/ingress/ingress-nginx/controller:v1.10.0#' deploy.yaml
sed -i 's#registry.k8s.io/ingress-nginx/kube-webhook-certgen:v1.4.0@sha256:44d1d0e9f19c63f58b380c5fddaca7cf22c7cee564adeff365225a5df5ef3334#repository.test.com:8444/kubernetes/ingress/ingress-nginx/kube-webhook-certgen:v1.4.0#' deploy.yaml

# create 创建
[root@k8s-master-1 ～]# kubectl create -f ingress-nginx-deploy.yaml 
namespace/ingress-nginx created
serviceaccount/ingress-nginx created
serviceaccount/ingress-nginx-admission created
role.rbac.authorization.k8s.io/ingress-nginx created
role.rbac.authorization.k8s.io/ingress-nginx-admission created
clusterrole.rbac.authorization.k8s.io/ingress-nginx created
clusterrole.rbac.authorization.k8s.io/ingress-nginx-admission created
rolebinding.rbac.authorization.k8s.io/ingress-nginx created
rolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx created
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
configmap/ingress-nginx-controller created
service/ingress-nginx-controller created
service/ingress-nginx-controller-admission created
deployment.apps/ingress-nginx-controller created
job.batch/ingress-nginx-admission-create created
job.batch/ingress-nginx-admission-patch created
ingressclass.networking.k8s.io/nginx created
validatingwebhookconfiguration.admissionregistration.k8s.io/ingress-nginx-admission created

# 查看状态
[root@k8s-master-1 test]# kubectl get pod -n ingress-nginx
NAME                                        READY   STATUS              RESTARTS   AGE
ingress-nginx-admission-create-wzd4v        0/1     Completed           0          11s
ingress-nginx-admission-patch-r8t8l         0/1     Completed           1          11s
ingress-nginx-controller-6fddd64f6b-fjhsn   1/1     Running             0          38s
```

## 三、Ingress-nginx 使用

**Ingress-nginx 三种配置方式**

1. **ConfigMap**：使用 ConfigMap 在 Nginx 中设置全局变量，设置后对所有 Ingress 规则生效。[官方文档说明](https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/configmap/)
2. **Annotations**：如果想要特定 Ingress 规则的配置，请使用此选项。[官方文档说明](https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/annotations/)
3. **自定义模板**：当需要更具体的设置时，例如 open_file_cache，在无法通过 ConfigMap 更改配置时调整临时监听选项。[官方文档说明](https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/custom-template/)

### 3.1、使用域名发布 k8s 服务

**创建一个 web 服务**

```bash
[root@localhost test]# kubectl create deploy nginx --image=10.202.43.191:5000/kubernetes/nginx/nginx:1.24.0
deployment.apps/nginx created
```

**暴漏服务**

```bash
[root@localhost test]# kubectl expose deploy nginx --port 80
service/nginx exposed
```

**创建 Ingress **

- **pathType**：路径的匹配方式，目前有`ImplementationSpecific`、`Exact`、`Prefix` 方式
  - **Exact**：精确匹配，比如配置的 path 为 /aaa，那么 /aaa/ 将不能被路由；
  - **Prefix**：前缀匹配，基于以 / 分割的 URL 路径。比如 path 为 /abc，可以匹配到 /abc/bbb 等，比较常用的配置；
  - **ImplementationSpecific**：这种类型的路由匹配根据 Ingress Controller 来实现，可以当作一个单独的类型，也可以当作 Prefix 和 Exact；是 1.18 版本引入 Prefix 和 Exact 的默认配置；

```bash
[root@localhost test]# cat web-ingress.yaml 
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx
spec:
  ingressClassName: nginx
  rules:
  - host: nginx.test.com
    http:
      paths:
      - path: /
        pathType: ImplementationSpecific
        backend:
          service:
            name: nginx
            port:
              number: 80
```

### 3.2、特例：不配置域名发布服务

```bash
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
      - path: /
        pathType: ImplementationSpecific
        backend:
          service:
            name: nginx
            port:
              number: 80
```

### 3.3、接口变化解析

1.19 版本之前的 v1beta1

```bash
apiVersion: networking.k8s.io/v1beta1   # 1.22 之前可以使用 v1beta1
kind: Ingress
metadata:
  name: nginx
  annotations:
    kubernetes.io/ingress.class: "nginx"  # 不同的 Controller，ingress.class 可能不一致
spec:
  ingressClassName: nginx
  rules:
  - host: nginx.test.com
    http:
      paths:
      - path: /abc
        pathType: Prefix
        backend:
          serviceName: nginx
          servicePort: 80
```

### 3.4、Redirect 域名重定向

官方文档：https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/annotations/#permanent-redirect

在 Nginx 作为代理服务器时，Redirect 可用于域名的重定向，比如访问 test.com 被重定向到 damo.com。Ingress 可以更简单的实现 Redirect 功能，

**示例**

- **permanent-redirect**：重定向到的域名；
- **permanent-redirect-code**：重定向代理，不配置默认 301（永久重定向，如果有其他原因可以自行配置）

```yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: test
  namespace: default
  annotations:
    kubernetes.io/ingress.class: "nginx"  #这里是ingress规则选用的那个ingress控制器，如果设置ingress-nginx为默认控制器可不写
    nginx.ingress.kubernetes.io/permanent-redirect: www.damo.com #重定向后的新域名
    nginx.ingress.kubernetes.io/permanent-redirect-code: '301'  #重定向http状态码
spec:
  rules:   
  - host: www.test.com
    http: 
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          serviceName: service-v1
          servicePort: 80
```

**验证**

```yaml
# 查看 ingress-nginx 的外部端口
[root@k8s-master-1 ～]# kubectl get svc -n ingress-nginx
NAME                                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx-controller             NodePort    172.10.237.113   <none>        80:31658/TCP,443:32028/TCP   115m
ingress-nginx-controller-admission   ClusterIP   172.10.214.39    <none>        443/TCP                      115m

# 查看创建的ingess
[root@k8s-master-1 ～]# kubectl get ing
Warning: extensions/v1beta1 Ingress is deprecated in v1.14+, unavailable in v1.22+; use networking.k8s.io/v1 Ingress
NAME   CLASS    HOSTS          ADDRESS        PORTS   AGE
test   <none>   www.test.com   10.202.43.55   80      13m

# 添加映射到 /etc/hosts
[root@k8s-master-1 test]# echo "10.202.43.55  www.test.com" >> /etc/hosts

# 访问测试
[root@k8s-master-1 ~]# curl test.com:31658 -I
HTTP/1.1 301 Moved Permanently
Date: Mon, 18 Mar 2024 15:59:15 GMT
Content-Type: text/html
Content-Length: 162
Connection: keep-alive
Location: www.damo.com
```

### 3.5、Rewrite 前后端分离

官方说明：https://kubernetes.github.io/ingress-nginx/examples/rewrite/

比如：现在有两个服务分别时 service-1 与 service-2  服务，假设 service-1 是前端服务，service-2 是后端福，访问 service-1 需要访问 /api-a，访问 service-2 需要访问 /api-b，两个服务的请求地址都是 /index.heml，如果不使用 Rewreite 则所有请求默认会返回 404。

**相关配置项**

`nginx.ingress.kubernetes.io/rewrite-target:` 重定向配置

**yaml 示例**

```yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: test-rewrite
  namespace: default
  annotations:
    kubernetes.io/ingress.class: "nginx"
    nginx.ingress.kubernetes.io/rewrite-target: "/$2"
spec:
  rules:
  - host: www.test.com
    http:
      paths:
      - pathType: Prefix
        path: "/api-a(/|$)(.*)"
        backend:
          serviceName: service-1
          servicePort: 80
      - pathType: Prefix
        path: "/api-a(/|$)(.*)"
        backend:
          serviceName: service-2
          servicePort: 80
```

### 3.6、错误代码重定向

这里的错误代码重定向分为全局设置与某个 Ingress 设置

**开启全局 Ingress 示例**

错误代码重定向功能需要使用helm重新部署ingress-nginx控制器部署步骤如下：

```bash
#修改valume.yml文件
defaultBackend:
  ## 
  enabled: true #开启这个功能                          
  image:
    repository: 192.168.10.254:5000/defaultbackend-amd64  #配置镜像
    tag: "1.5"
#更新ingress
[16:54:47 root@nexus ingress-nginx]#helm upgrade -n ingress-nginx ingress-nginx .
#验证是否生成一个新Pod
[16:55:28 root@nexus ingress-nginx]#kubectl get pod -n ingress-nginx 
NAME                                            READY   STATUS    RESTARTS   AGE
ingress-nginx-controller-nbxgf                  1/1     Running   0          17s
ingress-nginx-defaultbackend-58d87944cb-jd5tj   1/1     Running   0          30s
```

更新完成后访问一个不存在的页面会跳转到Error Server中的页面。

```bash
[16:56:34 root@node-1 ingress]#curl http://www.zhangzhuo.com/
default backend - 404
```

**设置某个ingress的错误重定向会覆盖全局设置，示例如下**

- nginx.ingress.kubernetes.io/default-backend: 默认后端service名称
- nginx.ingress.kubernetes.io/custom-http-errors: 那些错误代理重定向到默认后端svc

```yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: zhangzhuo-v1
  namespace: default
  annotations:
    kubernetes.io/ingress.class: "nginx"
    nginx.ingress.kubernetes.io/default-backend: 'service-v2'
    nginx.ingress.kubernetes.io/custom-http-errors: "404,415"
spec:
  rules:   
  - host: www.zhangzhuo.com
    http: 
      paths:
      - pathType: Prefix
        path: "/a"
        backend:
          serviceName: service-v1
          servicePort: 80
```

### 3.7、使用ssl加密

生产环境对外的服务，一般需要配置 HTTPS 协议，使用 Ingress 也可以非常方便的添加 https 证书。我们测试环境，可以使用 openssl 生成一个测试证书。如果是生产环境，证书为在第三方公司购买的证书，无需自行生成

```bash
# 生成证书
[root@localhost ingress]# openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout com.key -out com.crt -subj "/CN=www.aaa.com"
Generating a 2048 bit RSA private key
...........................................+++
.............................+++
writing new private key to 'com.key'
-----
[root@localhost ingress]# openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout org.key -out org.crt -subj "/CN=www.bbb.org"
Generating a 2048 bit RSA private key
....................+++
.....................+++
writing new private key to 'org.key'
-----
# 创建证书 secret
[root@localhost ingress]# kubectl create secret tls www.aaa.com --cert=com.crt --key=com.key -n software
secret/www.aaa.com created
[root@localhost ingress]# kubectl create secret tls www.bbb.org --cert=org.crt --key=org.key -n software
secret/www.bbb.org created
```

**添加 ingress 的 TLS 配置**

如果在 ingress 中设置 tls，即默认的 http 会被强制重定向到 https，http 不可访问，如果需要 http 与https 同时可以使用，需要设置 `nginx.ingress.kubernetes.io/ssl-redirect: "false"`，禁用强制重定向，这样可以同时使用 http 与 https

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: test
  namespace: software
  annotations:
    kubernetes.io/ingress.class: "nginx"
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
spec:
  rules:   
  - host: www.aaa.com
    http: 
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: nginx
            port:
              number: 80
  - host: www.bbb.org
    http: 
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: gogs
            port:
              number: 3000
  tls:
  - hosts:
    - www.aaa.com  #证书所授权的域名列表
    - www.bbb.org
    secretName: www.aaa.com #证书的Secret名字,自己匹配
    secretName: www.bbb.org
```

