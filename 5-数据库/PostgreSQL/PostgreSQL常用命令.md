### 查看所有库

```postgresql
\l
```

### 查看所有表

```postgresql
\dt
```

### 切换数据库

```postgresql
\c DATABASE_NAME;
```

### 清空数据库

```bash
//删除public模式以及模式里面所有的对象
DROP SCHEMA public CASCADE;
//创建public模式
CREATE SCHEMA public;
```

### 查看当前模式

```postgresql
show search_path;
```

### 查看所有SCHEMA

```pos
\dn
```

- 该查询结果返回一个结果集，其中包含数据库中所有模式的名称，**注意：这将返回包括系统模式在内的所有模式**

```sql
SELECT schema_name FROM information_schema.schemata;
```

- 如果只想查看用户创建的模式，则使用一下查询

```sql
SELECT schema_name FROM information_shcema.schemata WHERE schema_owner != 'postgres';
```

- 查看特定模式中的所有表或其他对象

```bash
SELECT table_name, table_type FROM information_shcema.tables WHERE table_schema = '要查看的模式名称';
```

## 创建用户和数据库

```
CREATE USER taskflow2 WITH PASSWORD 'taskflow2';    # 创建用户并设置密码
CREATE DATABASE taskflow2 with ENCODING = 'UTF8';   # 创建数据库并设置字符集
GRANT ALL PRIVILEGES ON database taskflow2 TO taskflow2;   # 给数据库授权
```

### 统计所有表得行数

```post
CREATE TYPE table_count AS (table_name TEXT, num_rows INTEGER); 

CREATE OR REPLACE FUNCTION count_em_all () RETURNS SETOF table_count  AS '
DECLARE 
    the_count RECORD; 
    t_name RECORD; 
    r table_count%ROWTYPE; 

BEGIN
    FOR t_name IN 
        SELECT 
            c.relname
        FROM
            pg_catalog.pg_class c LEFT JOIN pg_namespace n ON n.oid = c.relnamespace
        WHERE 
            c.relkind = ''r''
            AND n.nspname = ''public'' 
        ORDER BY 1 
        LOOP
            FOR the_count IN EXECUTE ''SELECT COUNT(*) AS "count" FROM '' || t_name.relname 
            LOOP 
            END LOOP; 

            r.table_name := t_name.relname; 
            r.num_rows := the_count.count; 
            RETURN NEXT r; 
        END LOOP; 
        RETURN; 
END;
' LANGUAGE plpgsql; 

SELECT
    * 
FROM
    count_em_all ( ) AS r 
ORDER BY
    r.num_rows DESC;
```

### 查看PG库的最大连接数

```postgresql
show max_connections;
```

### 查看PG库的当前连接数

```postgresql
SELECT "count"(*) FROM pg_catalog.pg_stat_activity;
select * from pg_stat_activity
```

### 修改PG库的最大连接数

```postgresql
alter system set max_connections= 2000
```

### 导入单张表


```shell
pg_dump -h 127.0.0.1 -U postgres -p 5432 -d DATEBASE -t tableNAME -f [保存路径和文件名]
```

### 删除连接、数据库、角色

```sql
select pg_terminate_backend(pid) from pg_stat_activity where DATNAME = 'template1';
select pg_terminate_backend(pg_stat_activity.pid) from pg_stat_activity where datname = 'hive_bak' and pid <> pg_backend_pid();
DROP DATABASE hive_bak;
DROP ROLE hive;
```

