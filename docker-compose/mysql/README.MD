# 创建目录
```
mkdir -p /data/docker/mysql8/{conf,data}
```

# 创建mysql配置文件
```
vim /data/docker/mysql8/conf/mysqld.cnf

[mysqld]
pid-file = /var/run/mysqld/mysqld.pid
socket = /var/run/mysqld/mysqld.sock
datadir = /var/lib/mysql
#default-time-zone = '+8:00'
max_connections = 1024
#log-error = /var/log/mysql/error.log
# By default we only accept connections from localhost
#bind-address = 127.0.0.1
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0
default-storage-engine=INNODB
max_allowed_packet=256M
character_set_server=utf8mb4
collation-server=utf8mb4_unicode_ci
innodb_default_row_format=DYNAMIC
innodb_log_file_size=1G
lower_case_table_names=1
#sql_mode=NO_AUTO_VALUE_ON_ZERO
sql_mode=STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION
gtid_mode=ON
enforce_gtid_consistency=ON
log_slave_updates=1
default_authentication_plugin=mysql_native_password

# binlog setting
slow_query_log="ON"
long_query_time=1
slow_query_log_file=/var/lib/mysql/mysql-slow.log
log-bin=/var/lib/mysql/mysql-bin
#enforce-gtid-consistency=ON
#gtid-mode=ON
binlog_format=ROW

server-id=1
```

# 创建docker-compose.yml配置文件
```
vim /data/docker/mysql8/docker-compose.yml

version: '3.7'

services:
    mysql:
        image: mysql:8.0
        restart: always
        ports:
            - "3306:3306"
        volumes:
            - /data/docker/mysql/data:/var/lib/mysql
            - /data/docker/mysql/conf:/etc/mysql/conf.d
            - /etc/localtime:/etc/localtime
        environment:
            - MYSQL_ROOT_PASSWORD=shangHai@123
```

# 启动MySQL服务
```
cd /data/docker/mysql8
docker-compose up -d
mysql -uroot -p -h127.0.0.1
```
