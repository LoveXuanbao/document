- secure_file_priv 为 NULL 时，表示限制mysqld不允许导入或导出。
- secure_file_priv 为 /tmp 时，表示限制mysqld只能在/tmp目录中执行导入导出，其他目录不能执行。
- secure_file_priv 没有值时，表示不限制mysqld在任意目录的导入导出。

查看 **secure_file_priv** 的值，默认为NULL，表示限制不能导入导出。

```shell
mysql> show global variables like '%secure_file_priv%';
+------------------+-------+
| Variable_name    | Value |
+------------------+-------+
| secure_file_priv | NULL  |
+------------------+-------+
1 row in set (0.00 sec)
```

因为 **secure_file_priv** 参数是只读参数，不能使用set global命令修改。

```shell
mysql> set global secure_file_priv='';
ERROR 1238 (HY000): Variable 'secure_file_priv' is a read only variable
```

## 解决办法

打开my.cnf 或 my.ini，加入以下语句后重启mysql。

```shell
secure_file_priv=''
```

查看secure_file_priv修改后的值

```shell
mysql> show global variables like '%secure_file_priv%';
+------------------+-------+
| Variable_name    | Value |
+------------------+-------+
| secure_file_priv |       |
+------------------+-------+
1 row in set (0.00 sec)
```