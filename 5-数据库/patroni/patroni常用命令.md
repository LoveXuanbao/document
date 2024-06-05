### 查看
```
patronictl -d etcd:http://10.200.64.42:2379 list postgres
```
### 切换
```
patronictl -d etcd:http://10.200.64.42:2379 switchover postgres
```
### 启动
```
systemctl start etcd
systemctl start patroni
```
### 集群修改最大连接数
```
curl -s -XPATCH -d '{"postgresql":{"parameters":{"max_connections":"2000"}}}' http://192.168.44.26:8008/config
```