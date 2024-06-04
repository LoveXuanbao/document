- Apollo详解

1. 简介：apollo(配置中心产品领域产品)是携程框架部研发并开源的一款生产级的配置中心产品，它能够集中管理应用在不同环境、不同集群的配置，配置修改后能够实时推送到应用端，并且具备规范的权限、流程治理等特性，适用于微服务配置管理场景。

git地址为：https://github.com/ctripcorp/apollo

2. 架构：Apollo采用分布式微服务架构

![image-20240308153316933](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20240308153316933.png)

总结：

1）ConfgService/AdminService/Client/Portal 是 Apollo 的四个核心微服务模块，相互协作完成配置中心业务功能，Eureka/MetaServer/NginxLB 是辅助微服务之间进行服务发现的模块。

2）Apollo 采用微服务架构设计，架构和部署都有一些复杂，但是每个服务职责单一，易于扩展。另外，Apollo 只需要一套 Portal 就可以集中管理多套环境 (DEV/FAT/UAT/PRO) 中的配置，这个是它的架构的一大亮点

3）服务发现是微服务架构的基础，在 Apollo 的微服务架构中，既采用 Eureka 注册中心式的服务发现，也采用 NginxLB 集中 Proxy 式的服务发现

4）如果不考虑分布式微服务架构中的服务发现问题，Apollo的最简架构如图所示：

![image-20240308153334189](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20240308153334189.png)

```
解释：1.ConfigService 是一个独立的微服务，服务于 Client 进行配置获取

2.Client 和 ConfigService 保持长连接，通过一种拖拉结合 (push & pull) 的模式，实现配置实时更新的同时，保证配置更新不丢失。

3.AdminService 是一个独立的微服务，服务于 Portal 进行配置管理。Portal 通过调用 AdminService 进行配置管理和发布

4.ConfigService 和 AdminService 共享 ConfigDB，ConfigDB 中存放项目在某个环境的配置信息

5.Protal 有一个独立的 PortalDB，存放用户权限、项目和配置的元数据信息。Protal 只需部署一份，它可以管理多套环境。
```

5）在4架构的基础上加上服务发现：

![image-20240308153351097](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20240308153351097.png)

```
解释：1.config/adminserver 启动后都会注册到eureka服务注册中心，并定期发送保活心跳

2.eureka采用集群方式部署，使用分布式一致性协议保证每个实例的状态最终一致
```

6）Eureka 是自带服务发现的 Java 客户端的，如果 Apollo 只支持 Java 客户端接入，不支持其它语言客户端接入的话，那么 Client 和 Portal 只需要引入 Eureka 的 Java 客户端，就可以实现服务发现功能。发现目标服务后，通过客户端软负载 (SLB，例如 Ribbon) 就可以路由到目标服务实例。

![image-20240308153404339](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20240308153404339.png)

7）因为应用场景不只是java还有其他语言，所以Apollo还引入metaserver角色，其实就是一个eureka的proxy，将 Eureka 的服务发现接口以更简单明确的 HTTP 接口的形式暴露出来，方便 Client/Protal 通过简单的 HTTPClient 就可以查询到 Config/AdminService 的地址列表。获取到服务实例地址列表之后，再以简单的客户端软负载 (Client SLB) 策略路由定位到目标实例，并发起调用。  也就是最开始的架构图

3.Apollo部署使用

基于华夏航空项目，Apollo部署是在k8s中部署。

准备工作：

两个同级目录下，以及下面的文件：

![image](https://gitlab.gridsum.com/techsupport-law/kb_for_deploy/uploads/62654b93a19577c35a38195c2600b187/image.png)

1.先部署Apollo-config-and-admin再部署Apollo-portal

修改Apollo-config-and-admin下面三个yaml文件的mysql地址以及镜像地址：

只需要修改下面文件的红框中的mysql地址即可：

![image-20240308153424361](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20240308153424361.png)

其余两个文件修改红框中的mysql地址以及信息即可：

![image-20240308153439580](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20240308153439580.png)

Apollo-protal目录下三个配置文件修改同上