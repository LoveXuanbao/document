# 修改密码

ES在安装的时候生成密码是通过auto自动生成的，想修改成我们自己指定的密码



```bash
su - elastic
cd /data/es-cluster/elasticsearch-7.16.1/bin
./elasticsearch-setup-passwords interactive
```

在指定生成密码的时候，报错如下信息

```bash
ERROR: Elasticsearch keystore file is missing [/data/es-cluster/elasticsearch-7.16.1/config/elasticsearch.keystore]
[elastic@gs-server-4545 bin]$ ./elasticsearch -d 
[elastic@gs-server-4545 bin]$ ./elasticsearch-setup-passwords interactive

Failed to authenticate user 'elastic' against http://10.202.43.46:9200/_security/_authenticate?pretty
Possible causes include:
 * The password for the 'elastic' user has already been changed on this cluster
 * Your elasticsearch node is running against a different keystore
   This tool used the keystore at /data/es-cluster/elasticsearch-7.16.1/config/elasticsearch.keystore


ERROR: Failed to verify bootstrap password
```

