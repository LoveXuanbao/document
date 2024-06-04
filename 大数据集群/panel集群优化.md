### 将 SSO 管理员设置为大数据集群管理员

```sql
insert into adminprivilege(privilege_id,permission_id,resource_id,principal_id) select (max(a.privilege_id)+1),1,1,b.principal_id from adminprivilege a, users b where b.user_name='admin' group by b.user_id;
```









## 将 /var/log 目录软连接到数据盘

前提：假设系统根目录分区小于 100G，并且没有单独的日志分区/var/log，那么我们应该将系统日志目录 /var/log 做软连接迁移到数据分区中，避免日志占满根分区导致系统故障

- 下面操作在主机上一台一台执行，并且全部执行完毕后，在panel页面上重启主机上所有组件

```
mkdir -p /data1/var/log

cp -rp /var/log/* /data1/var/log/

ll /data1/var/log/

rm -rf /var/log

ln -s /data1/var/log /var/log
```

![image-20231013103059400](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20231013103059400.png)