###  禁用本地无密码登录

- 找到 `MongoDB` 安装目录，打开 `mongodb.conf` 文件，找到或添加一下代码

```bash
security:
 authorization: enabled
```



### mongodb 备份命令

```shell
/usr/bin/mongodump -h 127.0.0.1:27017 -uadmin -p123123123 --authenticationDatabase admin  -d moebius -o /opt/mongod
```

### mongodb 恢复命令

```shell
mongorestore -h IP --port 端口 -u 用户名 -p 密码 -d 数据库 --drop 文件存在路径
[root@localhost mongodb]# mongorestore -d test /home/mongodb/test #test这个数据库的备份路径
mongorestore -h127.0.0.1 --port 27017 -uadmin -p123123123 --authenticationDatabase admin -d moebius admin2/moebius/

```

### mongodb 表导出命令，必须选择表

```shell
mongoexport -h 127.0.0.1:27017 -uadmin -p123123123 --authenticationDatabase admin -d moebius -c clusters -o /data/bk/clusters.json
```

### mongodb 表导入命令

```shell
mongoimport -h 127.0.0.1:27017 -uadmin -p123123123 -d moebius -c clusters --drop /data/bk/clusters.json
```

### 数据库修改密码

- 登录MongoDB所在的主机执行如下命令

```shell
mongo -u admin -p ## 连接MongoDB Server
use admin  ## 选择 admin 数据库
db.auth("admin","123123123")  ## 旧密码登录
db.changeUserPassword("admin","New_Password")  ## 修改密码
db.auth("admin","New_Passwork")  ## 使用新密码登录
```

### 查看集群状态

```bash
rs.status()
```

