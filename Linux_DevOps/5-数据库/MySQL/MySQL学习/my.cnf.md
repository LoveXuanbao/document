## mysql-bin-xxx设置过期时间并自动删除

## 问题描述：

- mysql集群，使用一段时间之后，发现磁盘使用率巨高

![](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/20230419172150.png)

- 经过排查，发现mysql本身数据量非常小，但是其中mysql-bin-xxx占用空间特别多，其中1.4T的空间全部是mysql-bin-xxx占用

![](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/20230419172217.png)

- 查看集群binlog过期时间，发现集群日志过期时间为0，表示所有的binlog日志永久都不会失效，不会自动删除

```sql
show variables like 'expire_logs_days';
select @@global.expire_logs_days;
```



![](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/20230419172308.png)

## 解决

处于节约空间考虑，binlog日志，只保留七天

修改配置文件 my.cnf文件

```ini
[mysqld]
expire_logs_days=7
max_binlog_size=500M
```

改文件设置之后不会立即清楚，触发的条件是 binlog日志大小超过我们设置 max_binlog_size

重新启动，手动触发生效

```sql
flush logs
```

如果binlog非常多，不要轻易设置该参数，有可能导致IO争用，可以使用purge命令进行清除

```sql
# 将 bin.xxxx之前的binlog清理掉
purge binary logs to 'bin.xxxxx';

# 将指定时间之前的清理掉
purge binary logs before '2023-04-01 23:59:59';

注意，不要轻易手动去删除binlog，会导致binlog.index和真实存在的binlog不匹配，而导致expire_logs_day失效
```

参考连接：https://www.shuzhiduo.com/A/KE5Q3QQjzL/