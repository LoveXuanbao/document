

### 1、数据备份

本次升级更新为热升级，为了保险起见，还是先进行数据备份工作，在进行升级

```bash
# 在mongodb数据库服务器上直接执行 mongo 命令即可进入到 mongo 命令行，紧接着使用管理员账号连接mongo数据库

> mongodb://username:password@xx.xx.xx.xx:27017   # 连接数据库
> show dbs       # 查看数据库

# 退出mongo命令行在主机命令行执行以下备份命令

mkdir /data/mongo_backend
# 备份命令格式
mongodump -h mongodb_IP:mongodb_port -d [数据库名] -o 备份目录
# 示例
mongodump -h 127.0.0.1:27017 -d aniutest -o /data/mongo_backend/
```

### 2、停止mongo数据库

```bash
systemctl stop mongod
cp /etc/mongod.conf /etc/mongod.conf_4.0.11.bak  # 备份原配置文件
```

### 3、下载4.0.13版本rpm包

```bash
wget https://repo.mongodb.org/yum/redhat/7/mongodb-org/4.0/x86_64/RPMS/mongodb-org-server-4.0.13-1.el7.x86_64.rpm
wget https://repo.mongodb.org/yum/redhat/7/mongodb-org/4.0/x86_64/RPMS/mongodb-org-mongos-4.0.13-1.el7.x86_64.rpm
wget https://repo.mongodb.org/yum/redhat/7/mongodb-org/4.0/x86_64/RPMS/mongodb-org-tools-4.0.13-1.el7.x86_64.rpm
wget https://repo.mongodb.org/yum/redhat/7/mongodb-org/4.0/x86_64/RPMS/mongodb-org-shell-4.0.13-1.el7.x86_64.rpm
wget https://repo.mongodb.org/yum/redhat/7/mongodb-org/4.0/x86_64/RPMS/mongodb-org-4.0.13-1.el7.x86_64.rpm
```

### 4、安装 4.0.13 版本

```bash
rpm -Fvh mongodb-org-server-4.0.13-1.el7.x86_64.rpm  --nodeps
rpm -Fvh mongodb-org-tools-4.0.13-1.el7.x86_64.rpm  --nodeps
rpm -Fvh mongodb-org-mongos-4.0.13-1.el7.x86_64.rpm  --nodeps
rpm -Fvh mongodb-org-shell-4.0.13-1.el7.x86_64.rpm --nodeps
rpm -Fvh mongodb-org-4.0.13-1.el7.x86_64.rpm
```

### 5、对比配置文件差异

如果有区别，请参考备份文件修改配置文件

```bash
diff /etc/mongod.conf /etc/mongod.conf.bak 
```

### 6、启动mongo服务

```bash
systemctl daemon-reload
systemctl restart mongod
```

### 7、数据恢复

查看数据库是否有丢数据，如果有，请使用以下命令把备份数据库导入到数据库中

```bash
# 命令恢复格式
mongorestore -h mongo_ip:mongo_port -d 要恢复的数据库名 导出的数据库目录
# 示例
mongorestore -h 127.0.0.1:27017 -d aniutest /data/mongo_backend/
```

