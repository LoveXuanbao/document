### 查看key登录缓存

```
klist
```

```bash
kadmin.local -q "listprincs"
```

###  删除key登陆缓存

```
kdestroy
```

### 删除pricipal

```
kadmin.local -q "delprinc hdfs-dongfenghuashen@DONGFENGHUASHEN.COM"
```

### 创建pricipal

```
kadmin.local -q "addprinc -randkey hdfs-dongfenghuashen@DONGFENGHUASHEN.COM"
```

### 生成keytab

```
kadmin.local -q "xst -k /etc/security/keytabs/hdfs.headless.keytab hdfs-dongfenghuashen@DONGFENGHUASHEN.COM"
```

### 查看keytab

```
klist -kt /etc/security/keytabs/hdfs.headless.keytab

# 返回结果
Keytab name: FILE:hdfs.headless.keytab
KVNO Timestamp           Principal
---- ------------------- ------------------------------------------------------
   3 10/12/2023 18:29:25 hdfs-dongfenghuashen@DONGFENGHUASHEN.COM
   3 10/12/2023 18:29:25 hdfs-dongfenghuashen@DONGFENGHUASHEN.COM
   3 10/12/2023 18:29:25 hdfs-dongfenghuashen@DONGFENGHUASHEN.COM
   3 10/12/2023 18:29:25 hdfs-dongfenghuashen@DONGFENGHUASHEN.COM
   3 10/12/2023 18:29:25 hdfs-dongfenghuashen@DONGFENGHUASHEN.COM
   3 10/12/2023 18:29:25 hdfs-dongfenghuashen@DONGFENGHUASHEN.COM
```

### 认证keytab

```
kinit -kt /etc/security/keytabs/hdfs.headless.keytab hdfs-dongfenghuashen@DONGFENGHUASHEN.COM
```







