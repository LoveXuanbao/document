```bash
Usage:
  pg_dump [OPTION]... [DBNAME] 数据库名放最后，不指定默认是系统变量PGDATABASE指定的数据库。


General options:(一般选项)

  -f, --file=FILENAME          导出后保存的文件名

  -F, --format=c|d|t|p         导出文件的格式(custom, directory, tar, plain, text(default))

  -j, --jobs=NUM               并行任务数

  -v, --verbose                详细信息

  -V, --version                版本信息

  -Z, --compress=0-9           压缩格式的压缩级别

  --lock-wait-timeout=TIMEOUT  在等待表锁超时后操作失败

  -?, --help                   帮助信息


Options controlling the output content:(控制输出的选项)

  -a, --data-only              只导出数据，不包括模式

  -b, --blobs                  在转储中包括大对象

  -c, --clean                  在重新创建之前，先清除（删除）数据库对象

  -C, --create                 在转储中包括命令,以便创建数据库（包括建库语句，无需在导入之前先建数据库）

  -E, --encoding=ENCODING      转储以ENCODING形式编码的数据

  -n, --schema=SCHEMA          只转储指定名称的模式

  -N, --exclude-schema=SCHEMA  不转储已命名的模式

  -o, --oids                   在转储中包括 OID

  -O, --no-owner               在明文格式中, 忽略恢复对象所属者

  -s, --schema-only            只转储模式, 不包括数据(不导出数据)

  -S, --superuser=NAME         在转储中, 指定的超级用户名

  -t, --table=TABLE            只转储指定名称的表

  -T, --exclude-table=TABLE    不转储指定名称的表

  -x, --no-privileges          不转储权限 (grant/revoke)

  --binary-upgrade             只能由升级工具使用

  --column-inserts             以带有列名的insert命令形式转储数据

  --disable-dollar-quoting     取消美元 (符号) 引号, 使用 SQL 标准引号

  --disable-triggers           在恢复数据的过程中禁用触发器

  --exclude-table-data=TABLE   不转储指定表的数据

  --inserts                    将数据转储为insert命令，而不是copy命令

  --no-security-labels         不分配安全标签进行转储

  --no-synchronized-snapshots  不在并行任务中使用同步快照

  --no-tablespaces             不转储表空间分配信息

  --no-unlogged-table-data     不转储未标记的表数据

  --quote-all-identifiers      引用所有标识符( 不是关键字 )

  --section=SECTION            转储命名部分(pre-data, data, or post-data)

  --serializable-deferrable    等待没有异常的情况下进行转储

  --use-set-session-authorization  使用SET SESSION AUTHORIZATION命令而不是ALTER OWNER命令来设置所有权


Connection options:(控制连接的选项)

  -d, --dbname=DBNAME      转储的数据库名

  -h, --host=HOSTNAME      数据库服务器的主机名或IP

  -p, --port=PORT          数据库服务器的端口号

  -U, --username=NAME      指定数据库的用户联接

  -w, --no-password        不显示密码提示输入口令

  -W, --password           强制口令提示 (自动)

  --role=ROLENAME          转储前设置角色


如果没有提供数据库名字, 那么使用 PGDATABASE 环境变量的数值.
```

