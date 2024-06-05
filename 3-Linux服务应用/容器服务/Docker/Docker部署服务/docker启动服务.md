## debian的docker安装

离线部署脚本`http://10.202.42.91:8081/repository/raw/docker/docker-install-v19.03.15.tar.gz`

解压脚本后修改以下参数执行即可`docker-install.sh`

- harbor：私有镜像仓库
- docker_data：docker数据目录
- docker_ins_dir：docker安装目录

## 1.gogs

镜像：`law-harbor.internal.gridsumdissector.com/demo/gogs:latest`

启动命令

- 注意挂载数据目录

```bash
docker run -it -d --name=gogs  --restart=always  -p 3000:3000 -v /data1/gogs:/data law-harbor.internal.gridsumdissector.com/demo/gogs:latest
```

## 2.zookeeper

镜像：`docker.gridsumdissector.com/library/bitnami/zookeeper:3.7.0`

启动命令

- ZOO_HEAP_SIZE为JVM内存限制单位M
- 注意挂载数据目录

```bash
docker run -it -d --name zookeeper --restart=always -p 2181:2181 -v /data1/zookeeper:/bitnami/zookeeper -e ALLOW_ANONYMOUS_LOGIN=yes -e ZOO_HEAP_SIZE=1024 docker.gridsumdissector.com/library/bitnami/zookeeper:3.7.0
```

## 3.kafka

镜像：`docker.gridsumdissector.com/library/wurstmeister/kafka:2.8.1`

启动命令

- KAFKA_ZOOKEEPER_CONNECT：为zookeeper连接地址端口
- KAFKA_HEAP_OPTS：JVM内存限制
- KAFKA_LISTENERS：监听的地址填宿主机地址
- 注意挂载数据目录
- 网络使用宿主机网络，无需端口映射

```bash
docker run -it -d --name kafka --restart=always --net host  -v /data1/kafka:/kafka -e KAFKA_ZOOKEEPER_CONNECT=10.201.60.26:2181 -e KAFKA_HEAP_OPTS="-Xmx2048m -Xms2048m" -e KAFKA_LISTENERS=PLAINTEXT://10.201.60.26:9092 -e ALLOW_PLAINTEXT_LISTENER=yes docker.gridsumdissector.com/library/wurstmeister/kafka:2.8.1
```

## 4.postgres

镜像：`docker.gridsumdissector.com/library/postgresql/postgresql:12.8`

启动命令

- POSTGRES_USER：超级管理员用户名
- POSTGRES_PASSWORD：超级管理员密码
- 注意挂载数据目录

```bash
docker run -it -d --restart=always --name postgres -p 5432:5432 -v /app/postgres:/var/lib/postgresql/data -e POSTGRES_USER=postgres -e POSTGRES_PASSWORD=TtVJTXptChlk28W3 harbor.zj.sgcc.com.cn/tznl/postgresql:12.8
```

## 5.es

镜像：`docker.gridsumdissector.com/gai/elasticsearch:7.16.1-ik-zj`

启动命令

- 注意挂载数据目录
- 注意修改vm.max_map_count=1000000内核参数

```bash
docker run -it -d --restart=always --name es -p 9200:9200 -v /app/es:/usr/share/elasticsearch/data -v /es-repo:/repo harbor.zj.sgcc.com.cn/tznl/elasticsearch:7.16.1-ik-zj-2
```

- 安装完成后需要登录es初始化账户密码，只需要第一次设置

```bash
docker exec -it es bash
cd bin/
./elasticsearch-setup-passwords interactive
```

## 6.minio

镜像：docker.gridsumdissector.com/library/minio:RELEASE.2021-08-20T18-32-01Z

启动命令

```bash
docker run -it -d --restart=always --name minio -e MINIO_ROOT_USER=minio -e MINIO_ROOT_PASSWORD=FynNd7jAo82OFHn1 -p 9000:9000 -p 9001:9001 -v /app/minio:/data harbor.zj.sgcc.com.cn/tznl/minio:RELEASE.2021-08-20T18-32-01Z server /data --console-address ":9001"
```

## 7.mysql

镜像：docker.gridsumdissector.com/library/mysql:5.7.35

启动命令

```bash
docker run -it -d --restart=always --name mysql -e MYSQL_ROOT_PASSWORD=123456 -e MARIADB_CHARACTER_SET=utf8 -e MARIADB_COLLATE=utf8_general_ci -p 3306:3306 -v /data1/mysql:/bitnami/mysql docker.gridsumdissector.com/library/mysql:5.7.35
```

## 8.nacos

```bash
docker run -it -d --name nacos -e SPRING_DATASOURCE_PLATFORM=mysql -e NACOS_REPLICAS=1 -e MYSQL_SERVICE_HOST=192.168.10.71 -e MYSQL_SERVICE_DB_NAME=nacos -e MYSQL_SERVICE_PORT=3306 -e MYSQL_SERVICE_USER=root -e MYSQL_SERVICE_PASSWORD=123456 -e MODE=standalone -e NACOS_SERVER_PORT=8848 -p 8848:8848 harbor.zj.sgcc.com.cn/tznl/nacos-server:2.0.0
```