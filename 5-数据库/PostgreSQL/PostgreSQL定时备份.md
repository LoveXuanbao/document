# 定时备份postgres全量数据库

> 本方法首先要先解决pg_dump和pg_dumoall命令在执行过程中不交互输入密码

## pg_dump命令行不输入密码的方法

- 在用户家目录下创建一个配置文件，提前将密码写入到这个配置文件中，格式如下

  ```
  hostname:port:database:username:password
  
  eg:
  192.168.88.100:5432:*:admin:FrB3aZnNAHf0K@7
  ```

- 需要将此文件放在`postgres`用户目录下，保存成`.pgpass`文件，并且权限为600

  ```
  su - postgres  ## 切换postgres用户
  
  ## 根据自己实际IP，端口，用户名密码写入配置文件
  echo "192.168.88.100:5432:*:admin:FrB3aZnNAHf0K@7" > .pgpass
  
  chmod 600 .pgpass       ## 修改权限
  
  ## 创建备份目录,根据自己实际情况，选择磁盘空间大的目录
  mkdir /data1/pgdata_bak
  
  ## 修改目录权限
  chown postgres.postgres /data1/pgdata_bak
  ```
  

## 全量备份并且保留10天

```
#!/bin/bash

su - postgres -c "/usr/local/pgsql/bin/pg_dumpall -h 10.153.11.84 -U admin -p 5432 > /data1/pgdata_bak/all_databases-`date +%Y-%m-%d-%H-%-M-%S`.sql"

sleep 60

## 查看备份目录下10天之前的文件，并删除
find /data1/pgdata_bak/*.sql -mtime +10 -exec rm {} \;
```

