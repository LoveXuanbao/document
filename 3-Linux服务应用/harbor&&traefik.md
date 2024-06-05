## 现状 && 背景
- **harbor** 服务端支持**http**方式和 **https**方式请求
- 当**harbor**使用**https**方式搭建时，docker客户端若想推拉镜像需要在客户段服务器上配置证书；
- 当**harbor**使用**http**方式搭建时，需要在docker server启动命令中增加（非安全模式访问镜像仓库）**--insecure-registry docker.image.registry**,然后重启docker server

## 需求 && 目标
- 根据显示情况当我们 **docker pull** 和 **docker push**官方镜像仓库时，我们既没有把官方仓库设置为非安全模式也未进行客户端证书认证
- 因此我们搭建的**harbor**需要做到在客户端不做任何配置的情况下，实现**docker pull**和**docker push**功能。

## 分析

- **docker client**不做任何配置的情况下,它发出的一定是**https**请求，所以我们的服务端必需要接收的也是**https**请求，然而客户端是不带证书的请求，这也就要求服务端不进行证书验证
- 仔细研究**harbor**服务端及其证书校验服务，发现Harbor使用的**nginx**提供的服务并使用nginx进行的ssl证书校验，校验通过后 **proxy_pass http**转发到后端服务（也就是说Harbor各个服务接收的都是Http请求，只是nginx进行了ssl证书校验）
- 然后研究nginx是否支持不校验客户端证书，经过长时间的配置修改尝试及翻阅资料查询，遗憾的是nginx并不支持服务器端不校验客户端证书,也就是证书是必需的；
- 这样情况下我们不得不走第二条路，寻找一个代理，支持服务器端不校验客户端证书，挡在(harbor,nginx)前边，这样就可以实现我们的目标

## 代理&&流程图

- 通过查询已知Jetty和Traefik均支持服务端不校验客户端证书的,鉴于我们已有Kubernetes Traefik环境只需创建个yaml即可完成测试（且对Jetty服务并不熟悉）;
- 使用Traefik代理harbor还有一个好处是当有多个harbor服务器时，可以做负载均衡；
- 我们搜索到了traefik代理harbor的文章，其架构如下所示；
```
Traefik(TLS 443)-->Nginx+Harbor(443)-->Harbor
```

- 从网上搜索的效果大部分是这样的架构图，实际上这并不能满足我们的需求；Traefik代理了https请求转发到Nginx，Traefik与Nginx交互是需要证书的，这里隐含了docker client需要带着证书请求，Traefik只是做了透传
- 我们进行调整，将架构改为如下：
```
Traefik(TLS 443)-->Nginx+Harbor(80)-->Harbor
```
- Traefik要开启InsecureSkipVerify = true （即服务端不校验客户端证书）这个配置对于Traefik是全局的