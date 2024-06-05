# PHPBU 数据库备份

# PHPBU 简介

   `phpbu `是一个`PHP`工具，可以创建和加密备份，将备份同步到其他服务器或云服务，并帮助您监视备份的创建

在[PHPBU网站上可以](https://phpbu.de/)深入连接所有功能并获得简单的“入门”教程。

​    `PHPBU` 是一个采用`PHP`对各种数据库如`ArangoDB`、`Directories`、`Elasticsearch`、`InfluxDB`、`Ldap`、`MongoDB`、`MySQL`、`Percona XtraBackup`、`PostgreSQL`、`Redis`等进行备份并支持备份加密及远程传输备份数据的备份工具。

​    [PHPBU Giehub](https://github.com/sebastianfeldmann/phpbu)

​    [PHPBU 官方资料](https://phpbu.de/documentation.html)

# 依赖需求

- Requirements
  - PHP >= 7.2
  - ext/curl
  - ext/dom
  - ext/json
  - ext/spl
- POSIX Shell
  - bzip2 or gzip
  - xz
  - zip

## 安装 PHP74 

1. 安装 `EPEL`源

   ```shell
   sudo yum install -y epel-release
   ```

2. 安装 Remi repo，即 remi-php74

   ```shell
   sudo yum install -y http://rpms.remirepo.net/enterprise/remi-release-7.rpm
   ```

3. 安装 `yum-utils`

   ```shell
   sudo yum install -y yum-utils
   ```

4. 启用 `remi rpeo` 

   ```shell
   sudo yum-config-manager --enable remi-php74
   sudo yum update
   ```

5. 安装 php 74

   ```shell
   sudo yum install php74 php74-php-cli php74-php-xml php74-php-mbstring php74-php-pgsql
   ```

## 安装phpbu （任选一种即可）

+ 方法一

  ```shell
  wget https://phar.phpbu.de/phpbu.phar
  chmod +x phpbu.phar
  php phpbu.phar --version
  mv phpbu.phar /usr/local/bin/phpbu
  ln -s /opt/remi/php74/root/usr/bin/php /usr/bin/php
  phpbu --version
  ```

+ 方法二

  1. 先安装`phive`

     ```shell
     wget -O phive.phar https://phar.io/releases/phive.phar
     wget -O phive.phar.asc https://phar.io/releases/phive.phar.asc
     gpg --keyserver pool.sks-keyservers.net --recv-keys 0x9D8A98B29B2D5D79
     gpg --verify phive.phar.asc phive.phar
     chmod +x phive.phar
     sudo mv phive.phar /usr/local/bin/phive
     ```

  2. 通过`phive`安装`phpbu`

     ```shell
     phive install phpbu
     ```

  3. 启动命令（可加入`crontab`进行周期任务）

     ```shell
     phpbu --configuration=phpbu.json
     ```

# PHP的使用

## 编辑 phpbu.json

<details>
    <summary>vim phpbu.json</summary>
    <code><pre>
{
  "verbose": true,
  "logging": [
    {
      "type": "mail",
      "options": {
        "recipients": "someone@example.com",
        "sendOnlyOnError": "false",
        "transport": "mail"
      }
    }
  ],
  "backups": [
    {
      "name": "DSS-Bakup",
      "source": {
        "type": "pgdump",
        "options": {
          "host": "10.29.97.93",
          "port": "5432",
          "database": "datascience",
          "user": "postgres",
          "password": "postgres"
        }
      },
      "target": {
        "dirname": "/data/pgbak/datascience",
        "filename": "pgdump-datascience-%Y%m%d-%H%i.sql",
        "compress": "bzip2"   //压缩格式可为 gzip、xz、bzip2、zip
      },
      "checks": [
        {
          "type": "SizeDiffPreviousPercent",
          "value": "10"
        }
      ],
      "cleanup": {
        "type": "Quantity",
        "options": {
          "amount": "3"
        }
      }
    },
    { 
      "name": "File_service-Backup",
      "source": {
        "type": "pgdump",
        "options": {
          "host": "10.29.97.93",
          "port": "5432",
          "database": "file_service",
          "user": "postgres",
          "password": "postgres"
        }
      },
      "target": {
        "dirname": "/data/pgbak/file_service",
        "filename": "pgdump-file_service-%Y%m%d-%H%i.sql",
        "compress": "bzip2"
      },
      "checks": [
        { 
          "type": "SizeDiffPreviousPercent",
          "value": "10"
        }
      ],
      "cleanup": {
        "type": "Quantity",
        "options": {
          "amount": "3"
        }
      }
    },
    {
      "name": "GVP-Datavisual-Bakup",
      "source": {
        "type": "pgdump",
        "options": {
          "host": "10.29.97.93",
          "port": "5432",
          "database": "gvp_datavisual",
          "user": "postgres",
          "password": "postgres"
        }
      },
      "target": {
        "dirname": "/data/pgbak/gvp",
        "filename": "pgdump-datavisual-%Y%m%d-%H%i.sql",
        "compress": "bzip2"
      },
      "checks": [
        {
          "type": "SizeDiffPreviousPercent",
          "value": "10"
        }
      ],
      "cleanup": {
        "type": "Quantity",
        "options": {
          "amount": "3"
        }
      }
    },
    {
      "name": "GVP-Dashboard-Bakup",
      "source": {
        "type": "pgdump",
        "options": {
          "host": "10.29.97.93",
          "port": "5432",
          "database": "gvp_dashboard",
          "user": "postgres",
          "password": "postgres"
        }
      },
      "target": {
        "dirname": "/data/pgbak/gvp",
        "filename": "pgdump-dashboard-%Y%m%d-%H%i.sql",
        "compress": "bzip2"
      },
      "checks": [
        {
          "type": "SizeDiffPreviousPercent",
          "value": "10"
        }
      ],
      "cleanup": {
        "type": "Quantity",
        "options": {
          "amount": "3"
        }
      }
    },
    {
      "name": "GVP-Externalwidget-Bakup",
      "source": {
        "type": "pgdump",
        "options": {
          "host": "10.29.97.93",
          "port": "5432",
          "database": "gvp_externalwidget",
          "user": "postgres",
          "password": "postgres"
        }
      },
      "target": {
        "dirname": "/data/pgbak/gvp",
        "filename": "pgdump-externalwidget-%Y%m%d-%H%i.sql",
        "compress": "bzip2"
      },
      "checks": [
        {
          "type": "SizeDiffPreviousPercent",
          "value": "10"
        }
      ],
      "cleanup": {
        "type": "Quantity",
        "options": {
          "amount": "3"
        }
      }
    },
    {
      "name": "GVP-Filecenter-Bakup",
      "source": {
        "type": "pgdump",
        "options": {
          "host": "10.29.97.93",
          "port": "5432",
          "database": "gvp_filecenter",
          "user": "postgres",
          "password": "postgres"
        }
      },
      "target": {
        "dirname": "/data/pgbak/gvp",
        "filename": "pgdump-filecenter-%Y%m%d-%H%i.sql",
        "compress": "bzip2"
      },
      "checks": [
        {
          "type": "SizeDiffPreviousPercent",
          "value": "10"
        }
      ],
      "cleanup": {
        "type": "Quantity",
        "options": {
          "amount": "3"
        }
      }
    },
    {
      "name": "GVP-Localdb-Bakup",
      "source": {
        "type": "pgdump",
        "options": {
          "host": "10.29.97.93",
          "port": "5432",
          "database": "gvp_localdb",
          "user": "postgres",
          "password": "postgres"
        }
      },
      "target": {
        "dirname": "/data/pgbak/gvp",
        "filename": "pgdump-localdb-%Y%m%d-%H%i.sql",
        "compress": "bzip2"
      },
      "checks": [
        {
          "type": "SizeDiffPreviousPercent",
          "value": "10"
        }
      ],
      "cleanup": {
        "type": "Quantity",
        "options": {
          "amount": "3"
        }
      }
    },
    {
      "name": "GVP-Rc-Bakup",
      "source": {
        "type": "pgdump",
        "options": {
          "host": "10.29.97.93",
          "port": "5432",
          "database": "gvp_rc",
          "user": "postgres",
          "password": "postgres"
        }
      },
      "target": {
        "dirname": "/data/pgbak/gvp",
        "filename": "pgdump-rc-%Y%m%d-%H%i.sql",
        "compress": "bzip2"
      },
      "checks": [
        {
          "type": "SizeDiffPreviousPercent",
          "value": "10"
        }
      ],
      "cleanup": {
        "type": "Quantity",
        "options": {
          "amount": "3"
        }
      }
    },
    { 
      "name": "keystone-Bakup",
      "source": {
        "type": "pgdump",
        "options": {
          "host": "10.29.97.93",
          "port": "5432",
          "database": "keystone",
          "user": "postgres",
          "password": "postgres"
        }
      },
      "target": {
        "dirname": "/data/pgbak/keystone",
        "filename": "pgdump-keystone-%Y%m%d-%H%i.sql",
        "compress": "bzip2"
      },
      "checks": [
        { 
          "type": "SizeDiffPreviousPercent",
          "value": "10"
        }
      ],
      "cleanup": {
        "type": "Quantity",
        "options": {
          "amount": "3"
        }
      }
    },
    { 
      "name": "knowledgebase-Bakup",
      "source": {
        "type": "pgdump",
        "options": {
          "host": "10.29.97.93",
          "port": "5432",
          "database": "knowledgebase",
          "user": "postgres",
          "password": "postgres"
        }
      },
      "target": {
        "dirname": "/data/pgbak/knowledgebase",
        "filename": "pgdump-knowledgebase-%Y%m%d-%H%i.sql",
        "compress": "bzip2"
      },
      "checks": [
        { 
          "type": "SizeDiffPreviousPercent",
          "value": "10"
        }
      ],
      "cleanup": {
        "type": "Quantity",
        "options": {
          "amount": "3"
        }
      }
    },
    { 
      "name": "kong-Bakup",
      "source": {
        "type": "pgdump",
        "options": {
          "host": "10.29.97.93",
          "port": "5432",
          "database": "kong",
          "user": "postgres",
          "password": "postgres"
        }
      },
      "target": {
        "dirname": "/data/pgbak/kong",
        "filename": "pgdump-kong-%Y%m%d-%H%i.sql",
        "compress": "bzip2"
      },
      "checks": [
        { 
          "type": "SizeDiffPreviousPercent",
          "value": "10"
        }
      ],
      "cleanup": {
        "type": "Quantity",
        "options": {
          "amount": "3"
        }
      }
    },
    { 
      "name": "license-server-Bakup",
      "source": {
        "type": "pgdump",
        "options": {
          "host": "10.29.97.93",
          "port": "5432",
          "database": "license-server",
          "user": "postgres",
          "password": "postgres"
        }
      },
      "target": {
        "dirname": "/data/pgbak/license-server",
        "filename": "pgdump-license-server-%Y%m%d-%H%i.sql",
        "compress": "bzip2"
      },
      "checks": [
        { 
          "type": "SizeDiffPreviousPercent",
          "value": "10"
        }
      ],
      "cleanup": {
        "type": "Quantity",
        "options": {
          "amount": "3"
        }
      }
    },
    { 
      "name": "moebius_alarm_center-Bakup",
      "source": {
        "type": "pgdump",
        "options": {
          "host": "10.29.97.93",
          "port": "5432",
          "database": "moebius_alarm_center",
          "user": "postgres",
          "password": "postgres"
        }
      },
      "target": {
        "dirname": "/data/pgbak/moebius",
        "filename": "pgdump-moebius_alarm_center-%Y%m%d-%H%i.sql",
        "compress": "bzip2"
      },
      "checks": [
        { 
          "type": "SizeDiffPreviousPercent",
          "value": "10"
        }
      ],
      "cleanup": {
        "type": "Quantity",
        "options": {
          "amount": "3"
        }
      }
    },
    {
      "name": "moebius_notice_center-Bakup",
      "source": {
        "type": "pgdump",
        "options": {
          "host": "10.29.97.93",
          "port": "5432",
          "database": "moebius_notice_center",
          "user": "postgres",
          "password": "postgres"
        }
      },
      "target": {
        "dirname": "/data/pgbak/moebius",
        "filename": "pgdump-moebius_notice_center-%Y%m%d-%H%i.sql",
        "compress": "bzip2"
      },
      "checks": [
        {
          "type": "SizeDiffPreviousPercent",
          "value": "10"
        }
      ],
      "cleanup": {
        "type": "Quantity",
        "options": {
          "amount": "3"
        }
      }
    },
    {
      "name": "Sso-Portal-Bakup",
      "source": {
        "type": "pgdump",
        "options": {
          "host": "10.29.97.93",
          "port": "5432",
          "database": "portal",
          "user": "postgres",
          "password": "postgres"
        }
      },
      "target": {
        "dirname": "/data/pgbak/sso",
        "filename": "pgdump-portal-%Y%m%d-%H%i.sql",
        "compress": "bzip2"
      },
      "checks": [
        {
          "type": "SizeDiffPreviousPercent",
          "value": "10"
        }
      ],
      "cleanup": {
        "type": "Quantity",
        "options": {
          "amount": "3"
        }
      }
    },
    {
      "name": "Project_test-Bakup",
      "source": {
        "type": "pgdump",
        "options": {
          "host": "10.29.97.93",
          "port": "5432",
          "database": "project_test",
          "user": "postgres",
          "password": "postgres"
        }
      },
      "target": {
        "dirname": "/data/pgbak/priject",
        "filename": "pgdump-project-test-%Y%m%d-%H%i.sql",
        "compress": "bzip2"
      },
      "checks": [
        {
          "type": "SizeDiffPreviousPercent",
          "value": "10"
        }
      ],
      "cleanup": {
        "type": "Quantity",
        "options": {
          "amount": "3"
        }
      }
    },
    {
      "name": "Sso-SingleSignOnLog-Bakup",
      "source": {
        "type": "pgdump",
        "options": {
          "host": "10.29.97.93",
          "port": "5432",
          "database": "SingleSignOnLog",
          "user": "postgres",
          "password": "postgres"
        }
      },
      "target": {
        "dirname": "/data/pgbak/sso",
        "filename": "pgdump-SingleSignOnLog-%Y%m%d-%H%i.sql",
        "compress": "bzip2"
      },
      "checks": [
        {
          "type": "SizeDiffPreviousPercent",
          "value": "10"
        }
      ],
      "cleanup": {
        "type": "Quantity",
        "options": {
          "amount": "3"
        }
      }
    },
    {
      "name": "Swift-Bakup",
      "source": {
        "type": "pgdump",
        "options": {
          "host": "10.29.97.93",
          "port": "5432",
          "database": "swift",
          "user": "postgres",
          "password": "postgres"
        }
      },
      "target": {
        "dirname": "/data/pgbak/swift",
        "filename": "pgdump-swift-%Y%m%d-%H%i.sql",
        "compress": "bzip2"
      },
      "checks": [
        {
          "type": "SizeDiffPreviousPercent",
          "value": "10"
        }
      ],
      "cleanup": {
        "type": "Quantity",
        "options": {
          "amount": "3"
        }
      }
    },
    {
      "name": "Sso-Tenant_console-Bakup",
      "source": {
        "type": "pgdump",
        "options": {
          "host": "10.29.97.93",
          "port": "5432",
          "database": "tenant",
          "user": "postgres",
          "password": "postgres"
        }
      },
      "target": {
        "dirname": "/data/pgbak/sso",
        "filename": "pgdump-tenant-%Y%m%d-%H%i.sql",
        "compress": "bzip2"
      },
      "checks": [
        {
          "type": "SizeDiffPreviousPercent",
          "value": "10"
        }
      ],
      "cleanup": {
        "type": "Quantity",
        "options": {
          "amount": "3"
        }
      }
    },
    {
      "name": "WebProxy-Bakup",
      "source": {
        "type": "pgdump",
        "options": {
          "host": "10.29.97.93",
          "port": "5432",
          "database": "WebProxy",
          "user": "postgres",
          "password": "postgres"
        }
      },
      "target": {
        "dirname": "/data/pgbak/webproxy",
        "filename": "pgdump-webproxy-%Y%m%d-%H%i.sql",
        "compress": "bzip2"
      },
      "checks": [
        {
          "type": "SizeDiffPreviousPercent",
          "value": "10"
        }
      ],
      "cleanup": {
        "type": "Quantity",
        "options": {
          "amount": "3"
        }
      }
    }
  ]
}
</pre></code>
</details>

## 邮件报警示例

![](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image.png)

# 还原验证

​    将备份文件解压后，使用`psql -f  backup.file`还原

  ```shell
psql -u USERNAME -p port -h IP地址 -d DATABASE_NAME -f backup.file
  ```
