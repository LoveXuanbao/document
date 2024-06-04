# MySQL的使用

## 密码管理

### 5.6 版本

- 5.6版本之前安装`mysql`之后，是没有密码的，需要先给默认用户`root`创建个密码，保证数据库的安全性，下面是给用户创建密码的语法
  
  - 创建密码
  
  ```shell
  /usr/bin/mysqladmin -u root password 'new-password'
  /usr/bin/mysqladmin -u root -h localhost password 'new-password'
  ```
  
  - 有密码修改密码
  
  ```shell
  mysql -u root -p123  password'456'
  ```
  
  - 忘记密码
  
    在配置文件/etc/my.cnf 的`[mysqld]`下添加`skip-grand-tables`，然后重启mysqld。注意，必须在`[mysqld]`下，并且修改完密码后，要把添加`skip-grand-tables`注释掉或者删除掉，在重启mysqld。
  
  ```shell
  vim /etc/my.cnf
  ...
  [mysqld]
  skip-grant-tables
  ...
  systemctl restart mysql
  //重启完成后输入mysql进入到sql界面
  update mysql.user set password=password'new_password' where user='root';
  flush privileges;
  
  exit  退出数据库
  
  ```
  
  然后把my.cnf刚才添加的skip-grand-tables注释掉，重启mysql即可使用新密码登录

### 5.7版本修改密码

MySQL由5.6升级至5.7版本后，MySQL5.7设置简易密码时，会直接报错，而且MySQL5.7中`mysql.user`表中的密码字段由之前的`password`改为`authentication_string`

MySQL 5.7 在初始安装后（CentOS 7.X 操作系统），会生成随机初始密码，并在`/var/log/mysqld.log`文件中有记录，可以查找并找password关键字，找到初始密码。

```shell
cat /var/log/mysqld.log | grep password
2020-05-13T09:26:13.833391Z 1 [Note] A temporary password is generated for root@localhost: ;+ze>hxNJ6i)
2020-05-13T09:26:27.305975Z 2 [Note] Access denied for user 'root'@'localhost' (using password: NO)
```

然后使用初始密码登录mysql

```shell
mysql -uroot -p';+ze>hxNJ6i)'
```

修改初始密码，只有先修改了初始密码，才能修改validate_password插件，然后修改密码为弱密码

```shell
alter user 'root'@'localhost' identified by 'qwER12#$';
或者
update mysql.user set authentication_string=password('root') where user='root';
```

MySQL5.7 设置简易密码是因为validate_password这插件导致的，通过`show variables like 'validate_password%'`就可以查看validate_password是否安装及其配置情况。

```shell
mysql> show variables like 'validate_password%';
+--------------------------------------+--------+
| Variable_name                        | Value  |
+--------------------------------------+--------+
| validate_password_check_user_name    | OFF    |
| validate_password_dictionary_file    |        |
| validate_password_length             | 3      |
| validate_password_mixed_case_count   | 0      |
| validate_password_number_count       | 3      |
| validate_password_policy             | MEDIUM |
| validate_password_special_char_count | 0      |
+--------------------------------------+--------+
```

- `validate_password_dictionary_file` ：插件用于验证密码强度的字典文件路径。
- `validate_password_length`： 密码最小长度。
- `validate_password_mixed_case_count` ：密码至少要包含的小写字母个数和大写字母个数。
- `validate_password_number_count`：密码至少要包含的数字个数。
- `validate_password_policy`：密码强度检查等级，0/LOW、1/MEDIUM、2/STRONG。
- `validate_password_special_char_count` ：密码至少要包含的特殊字符数

#### 方法一

```shell
数据库中执行
//验证密码混合情况计数
set global validate_password_mixed_case_count=0; 
 
//验证密码的长度
set global validate_password_number_count=3; 
 
//全局验证密码特殊字符计数
set global validate_password_special_char_count=0; 
 
//全局验证密码长度
set global validate_password_length=3;

//验证密码安全级别
set global validate_password_policy=LOW;
```

然后就可以修改免密为弱密码了

```shell
mysql> update mysql.user set authentication_string=password('root') where user='root';
Query OK, 1 row affected, 1 warning (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 1

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)

mysql> exit
Bye
[root@bogon mysql]# mysql -uroot -proot
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 7
Server version: 5.7.30 MySQL Community Server (GPL)

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 
```

#### 方法二

关闭密码强度插件`validate_password`

在MySQL配置文件`/etc/my.cnf`总的mysqld下面添加 `validate_password=OFF`

```shell
vim /etc/my.cnf
...
[mysqld]
validate_password=OFF
...
// 然后重启mysql
systemctl restart mysqld

// 以密码的登录到mysql，新装的话，获取初始密码登录mysql，已经有密码的，就使用源密码登录
// 获取初始密码
cat /var/log/mysqld.log | grep password

// 以密码登录
mysql -uroot -p
Enter password:      ##输入密码

// 修改初始密码
alter user 'root'@'localhost' identified by 'root';

// 退出登录，以新密码登录
mysql -uroot -proot
```

## 查看登录

`processlist`命令的输出结果显示了有哪些线程在运行，不仅可以查看当前所有的连接数，还可以查看当前的连接状态帮助识别除有问题的查询语句等。

如果是root账号，能看到所有用户的当前连接。如果是其他普通账号，则只能看到自己占用的连接。`show processslist`只能列出当前100条。如果想全部列出，则可以使用`show full processlist`命令

- `show processlist;`

```shell
mysql> show processlist;
+----+------+-----------+------+---------+------+----------+------------------+
| Id | User | Host      | db   | Command | Time | State    | Info             |
+----+------+-----------+------+---------+------+----------+------------------+
|  7 | root | localhost | NULL | Query   |    0 | starting | show processlist |
|  8 | root | localhost | NULL | Sleep   |    7 |          | NULL             |
+----+------+-----------+------+---------+------+----------+------------------+
2 rows in set (0.00 sec)
```

各个列的含义

1. `id列`：用户登录mysql时，系统分配的"connection_id"，可以使用函数connection_id()查看
2. `user列`：显示当前用户。如果不是root，这个命令就只显示用户权限范围的sql语句
3. `host列`：显示这个语句是从哪个ip的哪个端口上发的，可以用来跟踪出现问题语句的用户
4. `db列`：显示这个进程目前连接的是哪个数据库
5. `command列`：显示当前连接的执行的命令，一般取值为休眠（sleep），查询（query），连接（connect）等
6. `time列`：显示这个状态持续的时间，单位是秒
7. `state列`：显示使用当前连接的sql语句的状态，很重要的列。state描述的是语句执行中的某一个状态。一个sql语句，以查询为例，可能需要经过copying to tmp table、sorting result、sending data等状态才可以完成
8. `info列`：显示这个sql语句，是判断问题语句的一个重要依据

> 在主从复制环境中，`show processlist`或`show full processlist`对于判断状态很有帮助。

- `show full processlist;`

```shell
mysql> show full processlist;
+----+------+-----------+------+---------+------+----------+-----------------------+
| Id | User | Host      | db   | Command | Time | State    | Info                  |
+----+------+-----------+------+---------+------+----------+-----------------------+
|  7 | root | localhost | NULL | Query   |    0 | starting | show full processlist |
|  8 | root | localhost | NULL | Sleep   |   61 |          | NULL                  |
+----+------+-----------+------+---------+------+----------+-----------------------+
2 rows in set (0.00 sec)
```

## 踢出登录

有时候我们在执行一条SQL语句，或者更改表结构时，由于这张表的数据量巨大，往往会在执行操作后卡住，然后这张表就会被锁住，这是，我们可以杀掉这个进程。

- `kill ID`

```shell
mysql> kill 8;
Query OK, 0 rows affected (0.01 sec)

mysql> show processlist;
+----+------+-----------+------+---------+------+----------+------------------+
| Id | User | Host      | db   | Command | Time | State    | Info             |
+----+------+-----------+------+---------+------+----------+------------------+
|  7 | root | localhost | NULL | Query   |    0 | starting | show processlist |
+----+------+-----------+------+---------+------+----------+------------------+
1 row in set (0.00 sec)
```

## 创建数据库

1. 在`/var/lib/mysql`下创建目录（一般不使用）

2. 使用SQL语句创建`create database DBNAME`

   ```shell
   mysql> create database db1;
   Query OK, 1 row affected (0.02 sec)
   
   mysql> show databases;
   +--------------------+
   | Database           |
   +--------------------+
   | information_schema |
   | db1                |
   | mysql              |
   | performance_schema |
   | sys                |
   +--------------------+
   5 rows in set (0.00 sec)
   ```

## 删除数据库

- 使用SQL语句`drop database DBNAME`

  ```shell
  mysql> drop database db1;
  Query OK, 0 rows affected (0.03 sec)
  
  mysql> show databases;
  +--------------------+
  | Database           |
  +--------------------+
  | information_schema |
  | mysql              |
  | performance_schema |
  | sys                |
  +--------------------+
  4 rows in set (0.00 sec)
  ```

## 查询当前所在数据库

- 使用SQL语句`select database();`，可以看到当前没有在任何一个数据库，因为是刚登陆`MySQL`，不会进入到任何一个数据

  ```shell
  mysql> select database();
  +------------+
  | database() |
  +------------+
  | NULL       |
  +------------+
  1 row in set (0.01 sec)
  ```

## 切换数据库

- 使用`use DBNAME`，切换使用的数据库

  ```shell
  mysql> show databases;
  +--------------------+
  | Database           |
  +--------------------+
  | information_schema |
  | db1                |
  | mysql              |
  | performance_schema |
  | sys                |
  +--------------------+
  5 rows in set (0.04 sec)
  
  mysql> use db1;
  Database changed
  mysql> select database();
  +------------+
  | database() |
  +------------+
  | db1        |
  +------------+
  1 row in set (0.00 sec)
  ```

## 创建表

创建表之前要先进入到要创建表的库，使用SQL语句`create tablesNAME`

- 语法

```
CREATE [TEMPORARY] TABLE [IF NOT EXISTS] tbl_name
create table tbl_name(字段1 数据类型，字段2 数据类型，...)
```

- example

```
create table t1(ID int(10) not null, NAME char(30) not null, age int(3) not null, sex enum('nan','nv') not null, hobby set('music','read','study','eat','dance','sports'));
```

## 查看表结构

- 使用 `desc tbl_name`

  ```shell
  mysql> desc t1;
  +-------+----------------------------------------------------+------+-----+---------+-------+
  | Field | Type                                               | Null | Key | Default | Extra |
  +-------+----------------------------------------------------+------+-----+---------+-------+
  | ID    | int(10)                                            | NO   |     | NULL    |       |
  | NAME  | char(30)                                           | NO   |     | NULL    |       |
  | age   | int(3)                                             | NO   |     | NULL    |       |
  | sex   | enum('nan','nv')                                   | NO   |     | NULL    |       |
  | hobby | set('music','read','study','eat','dance','sports') | YES  |     | NULL    |       |
  +-------+----------------------------------------------------+------+-----+---------+-------+
  5 rows in set (0.05 sec)
  ```

## MySQL表数据的增删改查

> 数据库通过插入、更新和删除等方式来改变表中的记录。插入数据是向表中插入新的记录，通过`insert`语句来是新。更新数据是改变表中已经存在的数据，使用`update`语句来是新。删除数据是删除表中不在使用的数据，通过`delete`语句来实现

### 查询表数据

- 语法：`select * from tbl_name;`

```shell
mysql> desc test;
+-------+-------------------------------------+------+-----+---------+-------+
| Field | Type                                | Null | Key | Default | Extra |
+-------+-------------------------------------+------+-----+---------+-------+
| ID    | int(11)                             | NO   |     | NULL    |       |
| NAME  | char(30)                            | NO   |     | NULL    |       |
| sex   | enum('man','woman')                 | NO   |     | NULL    |       |
| age   | int(10)                             | NO   |     | NULL    |       |
| phone | char(12)                            | YES  |     | NULL    |       |
| hobby | set('music','read','dance','spoet') | YES  |     | NULL    |       |
+-------+-------------------------------------+------+-----+---------+-------+
6 rows in set (0.01 sec)

mysql> select * from test;
Empty set (0.00 sec)
```

### 插入表数据

插入数据是向表中插入新的记录，通过这种方式可以为表中增加新的数据。MySQL中，通过`insert`语句来插入新的数据。使用`insert`语句可以同时为表的所有字段插入数据，也可以为表的指定字段插入数据。`insert`语句可以同时插入多条记录，还可以将一个表中查询出来的数据插入到另一个表中。

通常情况下，插入的新记录要包含表的所有字段。INSERT语句有两种方式可以同时为表的所有字段插入数据。第一种方式是不指定具体的字段名。第二种方式是列出表的所有字段。

#### 为表的所有字段插入数据

1. `insert`语句中，不指定具体的字段名，其基本语句形式如下
   - 语法：`insert into tbl_name values(值1,值2,...值n);`


```shell
mysql> insert into test values(100,'xiaoming','man',10,'13188888888','music');
Query OK, 1 row affected (0.01 sec)

mysql> select * from test;
+-----+----------+-----+-----+-------------+-------+
| ID  | NAME     | sex | age | phone       | hobby |
+-----+----------+-----+-----+-------------+-------+
| 100 | xiaoming | man |  10 | 13188888888 | music |
+-----+----------+-----+-----+-------------+-------+
1 row in set (0.00 sec)
```

2. `insert`语句中列出所有字段，其基本语句形式如下
   - 语法：`insert into tbl_name(字段1,字段2,...字段n) values(值1,值2,...值n);`
   - 注意：字段名称可以还顺序

```shell
mysql> insert into test(NAME,age,phone,ID,sex,hobby) values('xiaohong','9','13588888888',101,'woman','music,dance');
Query OK, 1 row affected (0.01 sec)

mysql> select * from test;
+-----+----------+-------+-----+-------------+-------------+
| ID  | NAME     | sex   | age | phone       | hobby       |
+-----+----------+-------+-----+-------------+-------------+
| 100 | xiaoming | man   |  10 | 13188888888 | music       |
| 101 | xiaohong | woman |   9 | 13588888888 | music,dance |
+-----+----------+-------+-----+-------------+-------------+
2 rows in set (0.00 sec)
```

#### 为表的指定字段插入数据

基本语句形式如下：

- 语法：`insert into tbl_name(字段1,字段2,...字段n) values(值1,值2,...值n);`
- 注意：因为是为表插入指定字段，那没插入的字段一定是要允许为空的，不然会以一个空白字符显示，插入的字段可为空也可不为空。字段的顺序可以任意排列

```
mysql> insert into test(ID,NAME,sex,age) values(102,'xiaohua','woman',11);
Query OK, 1 row affected (0.00 sec)

mysql> select * from test;
+-----+----------+-------+-----+-------------+-------------+
| ID  | NAME     | sex   | age | phone       | hobby       |
+-----+----------+-------+-----+-------------+-------------+
| 100 | xiaoming | man   |  10 | 13188888888 | music       |
| 101 | xiaohong | woman |   9 | 13588888888 | music,dance |
| 102 | xiaohua  | woman |  11 | NULL        | NULL        |
+-----+----------+-------+-----+-------------+-------------+
3 rows in set (0.00 sec)
```

#### 同时插入多条记录

同时插入多条记录是指一个`insert`语句插入多条记录。当用户需要插入好几条记录，用户可以使用上面的两个方法逐条插入记录。但是，每次都要些一个新的`insert`语句。这样比较麻烦，MySQL中，一个`insert`语句可以同时插入多条记录。其基本语法形式如下：

- 语法：`insert into tbl_name[(字段列表)] values(取值列表1),(取值列表2),...(取值列表n);`

```shell
mysql> insert into test(ID,NAME,sex,age,phone,hobby)
    -> values(103,'xiaobai','man',10,null,'spoet'),
    -> (104,'xiaohei','man',12,'18888888888','spoet'),
    -> (105,'xiaohuang','woman',11,'18588888888','dance');
Query OK, 3 rows affected (0.00 sec)
Records: 3  Duplicates: 0  Warnings: 0

mysql> select * from test;
+-----+-----------+-------+-----+-------------+-------------+
| ID  | NAME      | sex   | age | phone       | hobby       |
+-----+-----------+-------+-----+-------------+-------------+
| 100 | xiaoming  | man   |  10 | 13188888888 | music       |
| 101 | xiaohong  | woman |   9 | 13588888888 | music,dance |
| 102 | xiaohua   | woman |  11 | NULL        | NULL        |
| 103 | xiaobai   | man   |  10 | NULL        | spoet       |
| 104 | xiaohei   | man   |  12 | 18888888888 | spoet       |
| 105 | xiaohuang | woman |  11 | 18588888888 | dance       |
+-----+-----------+-------+-----+-------------+-------------+
6 rows in set (0.00 sec)
```

#### 将查询结果插入到表中

`insert`语句可以将一个表中查询出来的数据插入到另一个表中。这样，可以方便不同表之间进行数据交换，其基本语法形式如下：

- 语法：`insert into tbl_name1(字段列表1) select 字段列表2 from tbl_name2 where 条件表达式;`

```shell
mysql> insert into test(ID,NAME,sex,age,phone,hobby)
    -> select ID,NAME,sex,age,phone,hobby
    -> from ceshi;
Query OK, 2 rows affected (0.01 sec)
Records: 2  Duplicates: 0  Warnings: 0

mysql> select * from ceshi;
+------+-----------+-------+-----+-------+-------+
| ID   | NAME      | sex   | age | phone | hobby |
+------+-----------+-------+-----+-------+-------+
| 1000 | xiaolan   | man   |  10 | NULL  | NULL  |
| 1001 | xiaocheng | woman |  11 | NULL  | NULL  |
+------+-----------+-------+-----+-------+-------+
2 rows in set (0.00 sec)

mysql> select * from test;
+------+-----------+-------+-----+-------------+-------------+
| ID   | NAME      | sex   | age | phone       | hobby       |
+------+-----------+-------+-----+-------------+-------------+
|  100 | xiaoming  | man   |  10 | 13188888888 | music       |
|  101 | xiaohong  | woman |   9 | 13588888888 | music,dance |
|  102 | xiaohua   | woman |  11 | NULL        | NULL        |
|  103 | xiaobai   | man   |  10 | NULL        | spoet       |
|  104 | xiaohei   | man   |  12 | 18888888888 | spoet       |
|  105 | xiaohuang | woman |  11 | 18588888888 | dance       |
| 1000 | xiaolan   | man   |  10 | NULL        | NULL        |
| 1001 | xiaocheng | woman |  11 | NULL        | NULL        |
+------+-----------+-------+-----+-------------+-------------+
8 rows in set (0.00 sec)
```

### 更新表数据

更新数据是更新表中已经存在的记录。通过这种方式可以改变表中已经存在的数据。MySQL中，通过`update`语句来更新数据。其基本语法形式如下：

- 语法：`update tbl_name set 字段名1=取值1,字段名2=取值2,...字段名n=取值n where 条件表达式;`

```shell
mysql> select * from ceshi;
+------+---------+-------+-----+-------+-------+
| ID   | NAME    | sex   | age | phone | hobby |
+------+---------+-------+-----+-------+-------+
| 1000 | xiaolan | man   |  10 | NULL  | NULL  |
| 1001 | xiaozi  | woman |  11 | NULL  | NULL  |
+------+---------+-------+-----+-------+-------+
2 rows in set (0.00 sec)

mysql> update ceshi set NAME='xiaoming' where ID=1001;
Query OK, 1 row affected (0.01 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> select * from ceshi;
+------+----------+-------+-----+-------+-------+
| ID   | NAME     | sex   | age | phone | hobby |
+------+----------+-------+-----+-------+-------+
| 1000 | xiaolan  | man   |  10 | NULL  | NULL  |
| 1001 | xiaoming | woman |  11 | NULL  | NULL  |
+------+----------+-------+-----+-------+-------+
2 rows in set (0.00 sec)
```

### 修改数据表

修改数据表是修改数据库中已经存在的数据表的结构。MySQL使用`alter table`语句修改表。常用的修改表操作有：修改名，修改字段名、修改字段数据类型、增加和删除字段、修改字段的排列位置、修改表的存储引擎、删除表的外键约束。

- 表结构调整语法：

```sehll
`alter table tal_name ...`
```

- 修改表名：`alter table tbl_name rename new_name`

```mysql
mysql> show tables;
+---------------+
| Tables_in_db1 |
+---------------+
| t1            |
+---------------+
1 row in set (0.00 sec)

mysql> alter table t1 rename test;
Query OK, 0 rows affected (0.04 sec)

mysql> show tables;
+---------------+
| Tables_in_db1 |
+---------------+
| test          |
+---------------+
1 row in set (0.00 sec)
```

- 增加字段：`alter table tbl_name add 字段名 数据类型...`


```mysql
mysql> desc test;
+-------+---------+------+-----+---------+-------+
| Field | Type    | Null | Key | Default | Extra |
+-------+---------+------+-----+---------+-------+
| ID    | int(10) | NO   |     | NULL    |       |
+-------+---------+------+-----+---------+-------+
1 row in set (0.00 sec)

mysql> alter table test add NAME char (20) not null;
Query OK, 0 rows affected (0.02 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> desc test;
+-------+----------+------+-----+---------+-------+
| Field | Type     | Null | Key | Default | Extra |
+-------+----------+------+-----+---------+-------+
| ID    | int(10)  | NO   |     | NULL    |       |
| NAME  | char(20) | NO   |     | NULL    |       |
+-------+----------+------+-----+---------+-------+
2 rows in set (0.00 sec)
```



- 修改数据类型：`alter table tbl_name modify 字段名 新的数据类型`


```mysq
mysql> desc test;
+-------+----------+------+-----+---------+-------+
| Field | Type     | Null | Key | Default | Extra |
+-------+----------+------+-----+---------+-------+
| ID    | int(10)  | NO   |     | NULL    |       |
| NAME  | char(20) | NO   |     | NULL    |       |
+-------+----------+------+-----+---------+-------+
2 rows in set (0.00 sec)

mysql> alter table test modify NAME varchar (30) not null;
Query OK, 0 rows affected (0.03 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> desc test;
+-------+-------------+------+-----+---------+-------+
| Field | Type        | Null | Key | Default | Extra |
+-------+-------------+------+-----+---------+-------+
| ID    | int(10)     | NO   |     | NULL    |       |
| NAME  | varchar(30) | NO   |     | NULL    |       |
+-------+-------------+------+-----+---------+-------+
2 rows in set (0.00 sec)
```



- 修改字段名：`alter table tbl_name change 原字段名 新字段名 新字段数据类型;`


```mysql
mysql> alter table test change NAME xingming char(20) not null;
Query OK, 0 rows affected (0.02 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> desc test;
+----------+----------+------+-----+---------+-------+
| Field    | Type     | Null | Key | Default | Extra |
+----------+----------+------+-----+---------+-------+
| ID       | int(10)  | NO   |     | NULL    |       |
| xingming | char(20) | NO   |     | NULL    |       |
+----------+----------+------+-----+---------+-------+
2 rows in set (0.00 sec)
```



- 删除字段：`alter table tbl_name drop 字段名;`


```mysql
mysql> desc test1;
+-------+------------+------+-----+---------+-------+
| Field | Type       | Null | Key | Default | Extra |
+-------+------------+------+-----+---------+-------+
| id    | int(11)    | YES  |     | NULL    |       |
| date  | bigint(11) | YES  |     | NULL    |       |
+-------+------------+------+-----+---------+-------+
2 rows in set (0.00 sec)

mysql> alter table test1 drop date;
Query OK, 0 rows affected (0.05 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> desc test1;
+-------+---------+------+-----+---------+-------+
| Field | Type    | Null | Key | Default | Extra |
+-------+---------+------+-----+---------+-------+
| id    | int(11) | YES  |     | NULL    |       |
+-------+---------+------+-----+---------+-------+
1 row in set (0.00 sec)
```




### 删除表数据

删除数据是删除表中已经存在的记录。通过这种方式可以删除不在使用的记录。MySQL中，使用`delete`语句来删除数据，其基本语法形式如下：

#### 方法一

- 语法：`delete from tbl_name [where 条件表达式]`
- 注意：删除表中的所有记录要慎重。

**删除符合条件的表数据**

```shell
mysql> select * from ceshi;
+------+----------+-------+-----+-------+-------+
| ID   | NAME     | sex   | age | phone | hobby |
+------+----------+-------+-----+-------+-------+
| 1000 | xiaolan  | man   |  10 | NULL  | NULL  |
| 1001 | xiaoming | woman |  11 | NULL  | NULL  |
+------+----------+-------+-----+-------+-------+
2 rows in set (0.00 sec)

mysql> delete from ceshi where ID=1001;
Query OK, 1 row affected (0.00 sec)

mysql> select * from ceshi;
+------+---------+-----+-----+-------+-------+
| ID   | NAME    | sex | age | phone | hobby |
+------+---------+-----+-----+-------+-------+
| 1000 | xiaolan | man |  10 | NULL  | NULL  |
+------+---------+-----+-----+-------+-------+
1 row in set (0.00 sec)
```

**删除表中所有数据**

```shell
mysql> select * from test;
+------+-----------+-------+-----+-------------+-------------+
| ID   | NAME      | sex   | age | phone       | hobby       |
+------+-----------+-------+-----+-------------+-------------+
|  100 | xiaoming  | man   |  10 | 13188888888 | music       |
|  101 | xiaohong  | woman |   9 | 13588888888 | music,dance |
|  102 | xiaohua   | woman |  11 | NULL        | NULL        |
|  103 | xiaobai   | man   |  10 | NULL        | spoet       |
|  104 | xiaohei   | man   |  12 | 18888888888 | spoet       |
|  105 | xiaohuang | woman |  11 | 18588888888 | dance       |
| 1000 | xiaolan   | man   |  10 | NULL        | NULL        |
| 1001 | xiaozi    | woman |  11 | NULL        | NULL        |
+------+-----------+-------+-----+-------------+-------------+
8 rows in set (0.00 sec)

mysql> delete from test;
Query OK, 8 rows affected (0.00 sec)

mysql> select * from test;
Empty set (0.00 sec)
```

#### 方法二

- 语法：`truncate tbl_name`

#### 两者区别

- 如果`delete`不加`where`子句，那么它和`truncate table`是一样的，但它们有一点不同，那就是`delete`可以返回被删除的记录数，而`truncate table`返回的是0。

- 如果一个表中有自增字段，使用truncate table和没有WHERE子句的delete删除所有记录后，这个自增字段将起始值恢复成1.如果你不想这样做的话，可以在`delete`语句中加上永真的WHERE，如`where 1`或`where true`。
- `delete from table1 where 1`;

- 上面的语句在执行时将扫描每一条记录。但它并不比较，因为这个WHERE条件永远为true。这样做虽然可以保持自增的最大值，但由于它是扫描了所有的记录，因此，它的执行成本要比没有`where`子句的delete大得多。

- 还有一点就是,如果要删除表中的所有数据,建议使用`truncate table`, 尤其是表中有大量的数据, 使用`truncate table`是将表结构重新建一次速度要比使用`delete from`快很多,而`delete from`是一行一行的删除,速度很慢.

### 删除表

删除已有的MySQL表是很容易的，但是要非常小心，因为删除了表，就无法恢复数据了

- 语法：`drop table tbl_name`

```shell
mysql> show tables;
+---------------+
| Tables_in_db1 |
+---------------+
| ceshi         |
| lianxi        |
| test          |
+---------------+
3 rows in set (0.00 sec)

mysql> drop table lianxi;
Query OK, 0 rows affected (0.01 sec)

mysql> show tables;
+---------------+
| Tables_in_db1 |
+---------------+
| ceshi         |
| test          |
+---------------+
2 rows in set (0.00 sec)
```



## MySQL 用户及权限管理

`user`：在MySQL中用户数据存放在mysql库的user表里，该表中保存了用户的用户名，密码和权限信息。

`db`：在MySQL中的mysql库的db表中，存放了用户仅仅对某个库的权限

`tables_priv`：在MySQL中的tables_priv表中，存放了用户仅仅对某一个表的权限

### 创建用户

- 命令：`create user 'username'@'host' [identified by 'password'];`

- 说明：
  - username：登录的用户名
  - password：用户的密码
  - host：指定可以登录的主机，其中localhost表示本机，%表示所有主机
  - identified by：以什么认证该用户
- 如果只使用用户名，那么MySQL会自动补全为：用户名@'%'

- 示例：

  - ```shell
    create table 'user1'@'%' indentified by '123';
    ```

  - 创建一个用户后，mysql.user表中会插入一行新的数据，但是该用户是没有任何权限的

### 删除用户

```
drop user 'username'@'host'
```

注意：删除用户的是时候，要精确匹配到mysql.user表的user和host字段，才能删除

### mysql.user 表说明

MySQL在安装时会自动创建一个名为`mysql`的数据库，`mysql`数据库中存储的都是用户权限表。用户登录以后，MySQL会根据这些权限表的内容为每个用户赋予相应的权限

`user`表是MySQL中最重要的一个权限表，用来记录允许连接到服务器的账号信息，需要注意的是，`在user表里启动的所有权限都是全局级的，适用于所有数据库`。

`user`表中的字段大致可以分为4类，分别是用户列、权限列、安全列和资源控制列。

#### 用户列

用户列存储了用户连接MySQL数据库时需要输入的信息。需要注意的是MySQL 5.7 版本不在使用 Password 来作为密码的字段，而改成了 `authentication_string`

MySQL 5.7 版本的用户列

| 字段名                | 字段类型 | 是否为空 | 默认值 | 说明   |
| --------------------- | -------- | -------- | ------ | ------ |
| Host                  | char(60) | NO       | 无     | 主机名 |
| User                  | char(32) | NO       | 无     | 用户名 |
| authentication_string | text     | YES      | 无     | 密码   |

用户登录时，如果这3个字段同时匹配，MySQL数据库系统才会允许登录。创建新用户时，也是设置这三个字段的值。修改用户密码时，实际就是修改`user`表的`authentication_string`字段的值。因此，这3个字段决定了用户是否能登录。

#### 权限

权限列的字段决定了用户的权限，用来描述在全局范围内允许对数据和数据库进行操作。

权限大致分为两大类，分别是高级管理权限和普通权限

- 高级管理权限主要对数据库进行管理，例如关闭服务的权限，超级权限和加载用户等
- 普通权限主要操作数据库，例如查询权限、修改权限等。

user 表的权限列包括 Select_priv、Insert_priv 等以 priv 结尾的字段，这些字段值的数据类型为 ENUM，可取的值只有 Y 和 N：Y 表示该用户有对应的权限，N 表示该用户没有对应的权限。从安全角度考虑，这些字段的默认值都为 N

| 权限                   | 字段类型      | 是否为空 | 默认值 | 权限级别               | 权限说明                                                     |
| :--------------------- | ------------- | -------- | ------ | :--------------------- | ------------------------------------------------------------ |
| Select_priv            | enum('N','Y') | NO       | N      | 表                     | 通过select命令查询数据权限                                   |
| Insert_priv            | enum('N','Y') | NO       | N      | 表                     | 插入数据权限                                                 |
| Update_priv            | enum('N','Y') | NO       | N      | 表                     | 更新数据的权限                                               |
| Delete_priv            | enum('N','Y') | NO       | N      | 表                     | 删除表数据的权限                                             |
| Create_priv            | enum('N','Y') | NO       | N      | 数据库或表             | 数据库、表、索引的创建权限                                   |
| Drop_priv              | enum('N','Y') | NO       | N      | 数据库或表             | 删除数据库、表权限                                           |
| Reload_priv            | enum('N','Y') | NO       | N      | 服务器管理             | 执行`flush`等让数据库重新`load`某些对象或者数据的命令的权限  |
| Shutdown_priv          | enum('N','Y') | NO       | N      | 服务器管理             | 关闭数据库权限                                               |
| Process_priv           | enum('N','Y') | NO       | N      | 服务器管理             | 查看进程权限                                                 |
| File_priv              | enum('N','Y') | NO       | N      | 服务器主机上的文件访问 | `load data infile` 和 `select ... info file` 权限            |
| Grant_priv             | enum('N','Y') | NO       | N      | 服务器管理             | 是否可以给其他用户赋予权限，默认只有root用户有此权限         |
| References_priv        | enum('N','Y') | NO       | N      | 表                     | 是否可以创建外键约束的权限                                   |
| Index_priv             | enum('N','Y') | NO       | N      | 表                     | 在已有表上对索引增删查权限                                   |
| Alter_priv             | enum('N','Y') | NO       | N      | 表                     | 表结构变更权限                                               |
| Show_db_priv           | enum('N','Y') | NO       | N      | 服务器管理             | 查看数据库权限，包括用户用于足够访问权限数据库               |
| Super_priv             | enum('N','Y') | NO       | N      | 服务器管理             | 是否可以执行某些强大的管理功能，例如执行`kill`线程，`change master`，`purge master logs`，`set global`等命令的权限 |
| Create_tmp_table_priv  | enum('N','Y') | NO       | N      | 服务器管理             | 临时表的创建权限                                             |
| Lock_tables_priv       | enum('N','Y') | NO       | N      | 服务器管理             | 执行`lock tables`命令显示给表加锁的权限                      |
| Execute_priv           | enum('N','Y') | NO       | N      | 存储过程               | 存储过程、函数、触发器的执行权限                             |
| Repl_slave_priv        | enum('N','Y') | NO       | N      | 服务器管理             | 复制环境中，`slave`连接用户所需要的复制权限，是否可以读取用于维护复制数据库环境的二进制日志文件 |
| Repl_client_priv       | enum('N','Y') | NO       | N      | 服务器管理             | 执行`show master status`和`show slave status`命令的权限，确定从服务器和主服务器的位置权限。 |
| Create_view_priv       | enum('N','Y') | NO       | N      | 视图                   | 创建视图权限                                                 |
| Show_view_priv         | enum('N','Y') | NO       | N      | 视图                   | 执行`show create view`命令查看view创建语句的权限             |
| Create_routine_priv    | enum('N','Y') | NO       | N      | 存储过程               | 存储过程、函数、触发器等的创建权限                           |
| Alter_routine_priv     | enum('N','Y') | NO       | N      | 存储过程               | 存储过程、函数、触发器等的变更权限                           |
| Create_user_priv       | enum('N','Y') | NO       | N      | 服务器管理             | 创建用户权限，是否可以执行`create user 'username';`命令      |
| Event_priv             | enum('N','Y') | NO       | N      | 服务器管理             | 创建、修改和删除事件的权限                                   |
| Trigger_priv           | enum('N','Y') | NO       | N      | 服务器管理             | 创建和删除触发器的权限                                       |
| Create_tablespace_priv | enum('N','Y') | NO       | N      | 服务器管理             | 创建表空间的权限                                             |
| usage                  | enum('N','Y') | NO       | N      | 服务器管理             | 特殊的“无权限”权限，新建用户后不授予任何权限的时候所拥有的最小权限 |

如果要修改权限，可以使用`GRANT`语句为用户赋予一些权限，也可以通过`UPDARA`语句更新`user表`的方式来设置权限

#### 安全列

安全列主要用来判断用户是否能够登录成功。

| 字段名                | 字段类型                          | 是否为空 | 默认值                | 说明                                                         |
| --------------------- | --------------------------------- | -------- | --------------------- | ------------------------------------------------------------ |
| ssl_type              | enum('','ANY','X509','SPECIFIED') | NO       |                       | 支持ssl标准加密安全字段                                      |
| ssl_cipher            | blob                              | NO       |                       | 支持ssl标准加密安全字段                                      |
| x509                  | blob                              | NO       |                       | 支持x509标准字段                                             |
| x509_subject          | blob                              | NO       |                       | 支持x509标准字段                                             |
| plugin                | char(64)                          | NO       | mysql_native_password | 引入plugins以进行用户连接时的密码验证，plugin创建外部/代理用户 |
| authentication_string | text                              | YES      |                       | 加密后的密码                                                 |
| password_expired      | enum('N','Y')                     | NO       | N                     | 密码是否过期（`N`未过期，`Y`已过期）                         |
| password_last_changed | timestamp                         | YES      |                       | 记录密码最近修改的时间                                       |
| password_lifetime     | password_lifetime                 | YES      |                       | 设置密码的有效使时间，单位为天数                             |
| account_locked        | enum('N','Y')                     | NO       | N                     | 用户是否被锁定（`Y`锁定，`N`未锁定）                         |

> 注意：即使password_expired为`Y`，用户也可以使用密码登录MySQL，但是不允许做任何操作。

通常标准的发型版不支持ssl，可以使用`show variables like 'have_openssl'`语句来查看是否具有`ssl`功能，如果 have_openssl 的值为disabled，那么则不支持 ssl 加密功能。

#### 资源控制列

资源控制列的字段用来限制用户使用的资源，user表中的资源控制列如下所示

| 字段名               | 字段类型         | 是否为空 | 默认值 | 说明                             |
| -------------------- | ---------------- | -------- | ------ | -------------------------------- |
| max_questions        | int(11) unsigned | NO       | 0      | 规定每小时允许执行查询的操作次数 |
| max_updates          | int(11) unsigned | NO       | 0      | 规定每小时允许执行更新的操作次数 |
| max_connections      | int(11) unsigned | NO       | 0      | 规定每小时允许执行的连接操作次数 |
| max_user_connections | int(11) unsigned | NO       | 0      | 规定允许同时建立的连接次数       |

以上字段的默认值为`0`，表示没有限制。一个小时内用户查询或者连接数量超过资源控制限制，用户将被锁定，直到下一个小时才可以再次执行对应的操作，可以使用`grant`语句更新这些字段的值。

### 权限管理

| 权限指定符 | 权限允许的操作                         |
| ---------- | -------------------------------------- |
| alter      | 修改表和索引                           |
| create     | 创建数据库和表                         |
| delete     | 删除表中已有的记录                     |
| drop       | 删除数据库和表                         |
| index      | 创建或删除索引                         |
| insert     | 向表中插入新行                         |
| select     | 检索表中的记录                         |
| update     | 修改现存表记录                         |
| file       | 读或写服务器上文件                     |
| process    | 查看服务器中执行的线程信息或杀死线程   |
| reload     | 重载授权表或清空日志、主机缓存或表缓存 |
| shutdown   | 关闭服务器                             |
| all        | 所有；all privileges同义词             |
| usage      | 特殊的“无权限”权限                     |

#### 查看用户权限

```
show grants for username;
```

`username`组成：`'username'@'主机IP或host或%'`；`%`代表所有主机

#### 赋予权限

```
grant 权限 on *.* to 'username'@'host';
```

- 权限：就是权限表的里以priv结尾的权限，此处不用写priv。
- `*.*`：前面第一个`*`表示库名，后面的`*`表示表名

> `grant`语句除了修改权限外还有两个小用法
>
> - 修改密码：`grant 权限 on 表名.库名 to '用户名'@‘host’ identified by '密码';`
> - 创建用户：上面的语句，如果没有该用户，则创建用户，添加密码，并且还赋予权限

在root用户给别的用户赋予ALL权限的时候，是不会赋予 grant 的权限的，如果想有 grant 的权限，在授权语句后面加上`with grant option`：

例如：`grant all on *.* to 'test'@'%' with grant option;`

#### 回收权限

```
revoke 权限 on 库名.表名 from ‘用户名’@‘host’;
```

### MySQL 安装安全化安装工具

```
mysql_secure_installation
```

## MySQL查询语句的用法

MySQL数据库使用 SQL select语句来查询数据。

### 语法

```shell
SELECT column_name,column_name
FROM table_name
[WHERE Clause]
[LIMIT N][ OFFSET M]
```

- 查询语句中可以使用一个或者多个表，表之间使用逗号(,)分隔，并使用`where`语句来设定查询条件。
- `select` 命令可以读取一条或者多条记录。
- 可以使用星号（*）来代替其他字段，SELECT语句会返回表的所有字段数据
- 可以使用`where`语句来包含任何条件。
- 可以使用`limit`属性来设定返回的记录数。
- 可以通过OFFSET指定`select`语句开始查询的数据偏移量。默认情况下偏移量为0。

创建一张测试表，并插入几条数据

```shell
mysql> create table test(
    -> id int(10),
    -> name char(20),
    -> yuwen int(10),
    -> shuxue int(10),
    -> yingyu int(10));
Query OK, 0 rows affected (0.02 sec)

mysql> insert into test values
    -> (1,'xiaoming',80,88,92),
    -> (2,'zhangsan',74,80,47),
    -> (3,'lisi',89,83,66),
    -> (4,'wangwu',80,82,66),
    -> (5,'maliu',90,98,99);
Query OK, 5 rows affected (0.00 sec)
Records: 5  Duplicates: 0  Warnings: 0

mysql> select * from test;
+------+----------+-------+--------+--------+
| id   | name     | yuwen | shuxue | yingyu |
+------+----------+-------+--------+--------+
|    1 | xiaoming |    80 |     88 |     92 |
|    2 | zhangsan |    74 |     80 |     47 |
|    3 | lisi     |    89 |     83 |     66 |
|    4 | wangwu   |    80 |     82 |     66 |
|    5 | maliu    |    90 |     98 |     99 |
+------+----------+-------+--------+--------+
5 rows in set (0.00 sec)
```

### 指定列查询

- 语句：`select 字段1,字段2,...字段n from tbl_name`

```shell
mysql> select id,name,shuxue from test;
+------+----------+--------+
| id   | name     | shuxue |
+------+----------+--------+
|    1 | xiaoming |     88 |
|    2 | zhangsan |     80 |
|    3 | lisi     |     83 |
|    4 | wangwu   |     82 |
|    5 | maliu    |     98 |
+------+----------+--------+
5 rows in set (0.00 sec)
```

### 去重查询

使用`disitinct`关键字，如果结果中有完全相同的行，就取出重复行

- 语句：`select distinct yuwen from test;`

```shell
mysql> select distinct yuwen from test;
+-------+
| yuwen |
+-------+
|    80 |
|    74 |
|    89 |
|    90 |
+-------+
4 rows in set (0.01 sec)
```

### 运算

- 数学运算：+ - * / %

- 比较运算： > < >= <= = !=

- 逻辑运算：&& || not

#### 数学运算

```shell
mysql> select 1+1;
+-----+
| 1+1 |
+-----+
|   2 |
+-----+
1 row in set (0.00 sec)

//计算三个列的总和
mysql> select id,name,yuwen+shuxue+yingyu from test;
+------+----------+---------------------+
| id   | name     | yuwen+shuxue+yingyu |
+------+----------+---------------------+
|    1 | xiaoming |                 260 |
|    2 | zhangsan |                 201 |
|    3 | lisi     |                 238 |
|    4 | wangwu   |                 228 |
|    5 | maliu    |                 287 |
+------+----------+---------------------+
5 rows in set (0.00 sec)
```

### 别名

计算之后以新的列名显示出来，使用`as NAME`

- 示例：`select id,name,yuwen+shuxue+yingyu as total from test`

```shell
mysql> select id,name,yuwen+shuxue+yingyu as total from test;
+------+----------+-------+
| id   | name     | total |
+------+----------+-------+
|    1 | xiaoming |   260 |
|    2 | zhangsan |   201 |
|    3 | lisi     |   238 |
|    4 | wangwu   |   228 |
|    5 | maliu    |   287 |
+------+----------+-------+
5 rows in set (0.00 sec)
```

### 排序

计算完总数之后，给总数大小排序，使用`order by`

- 升序，`order by 列名`

```shell
mysql> select id,name,yuwen+shuxue+yingyu as total 
    -> from test order by total;
+------+----------+-------+
| id   | name     | total |
+------+----------+-------+
|    2 | zhangsan |   201 |
|    4 | wangwu   |   228 |
|    3 | lisi     |   238 |
|    1 | xiaoming |   260 |
|    5 | maliu    |   287 |
+------+----------+-------+
5 rows in set (0.00 sec)
```

- 降序  `order by 列名 desc`

```shell
mysql> select id,name,yuwen+shuxue+yingyu as total 
    -> from test order by total desc;
+------+----------+-------+
| id   | name     | total |
+------+----------+-------+
|    5 | maliu    |   287 |
|    1 | xiaoming |   260 |
|    3 | lisi     |   238 |
|    4 | wangwu   |   228 |
|    2 | zhangsan |   201 |
+------+----------+-------+
5 rows in set (0.00 sec)
```

### 显示N条记录

- 语句：`limit NUM`

```shell
mysql> select id,name,yuwen+shuxue+yingyu as total 
    -> from test order by total desc limit 3;
+------+----------+-------+
| id   | name     | total |
+------+----------+-------+
|    5 | maliu    |   287 |
|    1 | xiaoming |   260 |
|    3 | lisi     |   238 |
+------+----------+-------+
3 rows in set (0.00 sec)
```

### where查询过滤

在`where`语句中有很多经常使用到的运算符，如下：

![](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/20180526155434349.png)

### 比较查询

1. 查询英语成绩大于90的同学

```shell
mysql> select id,name,yingyu from test
    -> where yingyu>90;
+------+----------+--------+
| id   | name     | yingyu |
+------+----------+--------+
|    1 | xiaoming |     92 |
|    5 | maliu    |     99 |
+------+----------+--------+
2 rows in set (0.00 sec)
```

2. 查询所有总分大于270分的同学

- 注意：where语句后不能使用别名，因为数据库中先执行where语句，在执行select语句。

```shell
mysql> select id,name,yuwen+shuxue+yingyu as total from test
    -> where yuwen+shuxue+yingyu>270;
+------+-------+-------+
| id   | name  | total |
+------+-------+-------+
|    5 | maliu |   287 |
+------+-------+-------+
1 row in set (0.00 sec)
```

3. 模糊查询，`like`

- 查询姓li并且id小于4的同学

```shell
mysql> select id,name from test
    -> where name like 'li%' and id<4;
+------+------+
| id   | name |
+------+------+
|    3 | lisi |
+------+------+
1 row in set (0.00 sec)
```

4. 查询语文成绩大于英语成绩的同学

```shell
mysql> select * from test where yuwen>yingyu;
+------+----------+-------+--------+--------+
| id   | name     | yuwen | shuxue | yingyu |
+------+----------+-------+--------+--------+
|    2 | zhangsan |    74 |     80 |     47 |
|    3 | lisi     |    89 |     83 |     66 |
|    4 | wangwu   |    80 |     82 |     66 |
+------+----------+-------+--------+--------+
3 rows in set (0.00 sec)
```

5. 查询所有总分大于200并且数学成绩大于语文成绩的同学

```shell
mysql> select *,yuwen+shuxue+yingyu from test
    -> where yuwen+shuxue+yingyu>220 and shuxue>yuwen;
+------+----------+-------+--------+--------+---------------------+
| id   | name     | yuwen | shuxue | yingyu | yuwen+shuxue+yingyu |
+------+----------+-------+--------+--------+---------------------+
|    1 | xiaoming |    80 |     88 |     92 |                 260 |
|    4 | wangwu   |    80 |     82 |     66 |                 228 |
|    5 | maliu    |    90 |     98 |     99 |                 287 |
+------+----------+-------+--------+--------+---------------------+
3 rows in set (0.00 sec)
```

6. 查询所有英语成绩在90到100分的同学

方法一

```shell
mysql> select id,name,yingyu from test
    -> where yingyu>90 and yingyu<100;
+------+----------+--------+
| id   | name     | yingyu |
+------+----------+--------+
|    1 | xiaoming |     92 |
|    5 | maliu    |     99 |
+------+----------+--------+
2 rows in set (0.00 sec)
```

方法二

关键字：between 是区间

```shell
mysql> select id,name,yingyu from test
    -> where yingyu between 90 and 100;
+------+----------+--------+
| id   | name     | yingyu |
+------+----------+--------+
|    1 | xiaoming |     92 |
|    5 | maliu    |     99 |
+------+----------+--------+
2 rows in set (0.00 sec)
```

7. 查询数学成绩为80、81、82的同学

   `or`也可以换成 `||`

```shell
mysql> select id,name,shuxue from test
    -> where shuxue=80 or shuxue=81 or shuxue=82;
+------+----------+--------+
| id   | name     | shuxue |
+------+----------+--------+
|    2 | zhangsan |     80 |
|    4 | wangwu   |     82 |
+------+----------+--------+
2 rows in set (0.00 sec)
```

或者使用`in`

```shell
mysql> select id,name,shuxue from test
    -> where shuxue in(80,81,82);
+------+----------+--------+
| id   | name     | shuxue |
+------+----------+--------+
|    2 | zhangsan |     80 |
|    4 | wangwu   |     82 |
+------+----------+--------+
2 rows in set (0.01 sec)
```

### 函数

1. `count()`：count(*)统计条数

```shell
mysql> select count(*) from test;
+----------+
| count(*) |
+----------+
|        5 |
+----------+
1 row in set (0.00 sec)
```

2. `sum()`：sum(列名)，统计改列的总数

```shell
mysql> select sum(shuxue) from test;
+-------------+
| sum(shuxue) |
+-------------+
|         431 |
+-------------+
1 row in set (0.01 sec)
```

3. `avg()`：avg(列名)，统计该列的平均值

```shell
mysql> select avg(yuwen) from test;
+------------+
| avg(yuwen) |
+------------+
|    82.6000 |
+------------+
1 row in set (0.00 sec)
```

或者

```shell
mysql> select sum(yuwen)/count(*) as pingjunzhi from test;
+------------+
| pingjunzhi |
+------------+
|    82.6000 |
+------------+
1 row in set (0.00 sec)
```

4. `min`和`max`：最大数和最小数

```shell
mysql> select min(yuwen) from test;
+------------+
| min(yuwen) |
+------------+
|         74 |
+------------+
1 row in set (0.01 sec)

mysql> select max(yuwen) from test;
+------------+
| max(yuwen) |
+------------+
|         90 |
+------------+
1 row in set (0.00 sec)
```

5. 查询全级，数学最高分和最低分的人

```shell
mysql> select name,shuxue from test where shuxue = (select max(shuxue) from test);
+-------+--------+
| name  | shuxue |
+-------+--------+
| maliu |     98 |
+-------+--------+
1 row in set (0.00 sec)

mysql> select name,shuxue from test where shuxue = (select min(shuxue) from test);
+----------+--------+
| name     | shuxue |
+----------+--------+
| zhangsan |     80 |
+----------+--------+
1 row in set (0.00 sec)
```



### 分组  group by

先在原有的表里添加班级信息，给同学分班

```shell
mysql> alter table test add class char(10);
Query OK, 0 rows affected (0.07 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> select * from test;
+------+----------+-------+--------+--------+-------+
| id   | name     | yuwen | shuxue | yingyu | class |
+------+----------+-------+--------+--------+-------+
|    1 | xiaoming |    80 |     88 |     92 | NULL  |
|    2 | zhangsan |    74 |     80 |     47 | NULL  |
|    3 | lisi     |    89 |     83 |     66 | NULL  |
|    4 | wangwu   |    80 |     82 |     66 | NULL  |
|    5 | maliu    |    90 |     98 |     99 | NULL  |
+------+----------+-------+--------+--------+-------+
5 rows in set (0.00 sec)

mysql> update test set class=1 where id=1 || id=2 || id=4;
Query OK, 3 rows affected (0.01 sec)
Rows matched: 3  Changed: 3  Warnings: 0

mysql> update test set class=2 where id=3 || id=5;
Query OK, 2 rows affected (0.00 sec)
Rows matched: 2  Changed: 2  Warnings: 0

mysql> select * from test;
+------+----------+-------+--------+--------+-------+
| id   | name     | yuwen | shuxue | yingyu | class |
+------+----------+-------+--------+--------+-------+
|    1 | xiaoming |    80 |     88 |     92 | 1     |
|    2 | zhangsan |    74 |     80 |     47 | 1     |
|    3 | lisi     |    89 |     83 |     66 | 2     |
|    4 | wangwu   |    80 |     82 |     66 | 1     |
|    5 | maliu    |    90 |     98 |     99 | 2     |
+------+----------+-------+--------+--------+-------+
5 rows in set (0.00 sec)
```

统计每个班级有多少人

```shell
mysql> select class,count(class) from test group by class;
+-------+--------------+
| class | count(class) |
+-------+--------------+
| 1     |            3 |
| 2     |            2 |
+-------+--------------+
2 rows in set (0.00 sec)
```

## MySQL容灾

### 备份及还原

#### 全备份

全备份可以选择一周一次，一天一次等。

全备份分为冷备份于热备份

- 冷备份：关闭MySQL服务，拷贝所有库表文件
- 热备份：不关闭MySQL服务，使用mysqldump

#### 冷备份

- 备份时间通过监控，查看每天哪个时间在线用户最少。尽量减小关闭MySQL服务造成的影响
- myisam表需要拷贝的文件：`tbl_name.frm`、`tbl_name.MYD`、`tbl_name.MYI`；拷贝完成之后，需要修改文件权限和所属者于所属组。
- innodb表需要拷贝的文件:`tbl_name.frm`、`tbl_name.ibd`

#### 热备份

```
mysqldump
```

#### 备份同一个数据库的多张表

- 语法：

  `mysqldump [OPTIONS] database [tables] > /备份路径/备份文件名` 

  备份同一个数据库的多张表

- [options]

  `-x`：在备份时加锁

  `--single-transaction`：备份innodb表的时候，建议使用该选项，不使用`-x`

- 例子：

  `mysqldump -x -u username -ppassword -h host database tbl_name1 tbl_name2 > /tmp/test.sql` 

- 还原

  `mysql -uroot -p1234 dbname < /tmp/test.sql`

```shell
OR     mysqldump [OPTIONS] --all-databases [OPTIONS]
```

#### 备份数据库

- 语法：可以跟多个数据库，一起备份到一个文件中

  `mysqldump [OPTIONS] --databases [OPTIONS] DB1 [DB2 DB3...] > /备份路径/备份文件名`

- 例子

  `mysqldump -x -uroot -p1234 --databases db1 > /tmp/db1.sql`

- 还原

  `mysql -u root -p1234 < /tmp/db1.sql`

#### 备份所有数据库

- 语法

  `mysqldump [OPTIONS] --all-databases [OPTIONS]`

  注意：`--all-databases`也可以写成` -A`

- 例子

  `mysqldump -x -uroot -p1234 --all-databases > /tmp/all.sql`

- 还原

  `mysql -uroot -p1234 < /tmp/all.sql`

#### 增量备份

找一个日志，将用户对数据库的增删改的操作以`sql`语句的形式记录下来。

这个日志称为：二进制日志（log_bin）

