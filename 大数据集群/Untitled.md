### 通过restful接口获取CDH集群配置信息

```
curl -u cm-user:cm-pwd "http://cm-host:7180/api/v19/cm/deployment"
```



### beeline连接

```
beeline
!connect jdbc:hive2://10.202.82.138:10000/default;principal=hive/gs-server-13403@TESTPASS.COM

!connect jdbc:hive2://zeta1:2181,zeta2:2181,zeta3:2181/default;serviceDiscoveryMode=zooKeeper;zooKeeperNamespace=hiveserver2;principal=hive/zeta1@TARZAN.COM


!connect jdbc:hive2://pawucdpmgt01:2181,pawucdpmgt02:2181,pawucdpcnd01:2181/default;serviceDiscoveryMode=zooKeeper;zooKeeperNamespace=hiveserver2;principal=hive/pawucdpmgt01@ZETAPROD.COM

!connect jdbc:hive2://pawucdpmgt01:2181,pawucdpmgt02:2181,pawucdpcnd01:2181/;serviceDiscoveryMode=zooKeeper;zooKeeperNamespace=hiveserver2;principal=datastudio/pawucdpcnd01@ZETAPROD.COM
```



### yarn查看任务日志

```bash
yarn logs -applicationId application_1707318342167_140274 -out /path/to/logs.log
yarn logs -applicationId application_1707318342167_140274 -containerId container_e52_1707318342167_140274_02_000001  -logFiles prelaunch.err -out /path/to/prelaunch.err
```



### yarn 杀死 application

```bash
yarn application -kill + application id
```

