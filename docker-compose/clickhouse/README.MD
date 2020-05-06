# 创建目录
```
mkdir /data/docker/clickhouse/{data,log,conf}
```

# 生成随机密码
```
PASSWORD=$(base64 < /dev/urandom | head -c8); echo "$PASSWORD"; echo -n "$PASSWORD" | sha256sum | tr -d '-'
```

# 创建用户的配置文件
```
vim /data/docker/clickhouse/conf/users.xml

<?xml version="1.0"?>
<yandex>
    <profiles>
        <default>
            <max_memory_usage>10000000000</max_memory_usage>
            <use_uncompressed_cache>0</use_uncompressed_cache>
            <load_balancing>random</load_balancing>
        </default>

        <readonly>
            <max_memory_usage>10000000000</max_memory_usage>
            <use_uncompressed_cache>0</use_uncompressed_cache>
            <load_balancing>random</load_balancing>
            <readonly>1</readonly>
        </readonly>

    </profiles>

    <quotas>
        <!-- Name of quota. -->
        <default>
            <interval>
                <duration>3600</duration>
                <queries>0</queries>
                <errors>0</errors>
                <result_rows>0</result_rows>
                <read_rows>0</read_rows>
                <execution_time>0</execution_time>
            </interval>
        </default>
    </quotas>

    <users>
        <root>
            <password_sha256_hex>42581fb927b0128282c1f409c8b396b49add775940c21b681e22869d9b153b8e</password_sha256_hex>
            <networks incl="networks" replace="replace">
                <ip>::/0</ip>
            </networks>
            <profile>default</profile>
            <quota>default</quota>
        </root>

        <read>
            <password_sha256_hex>e209335e37ac55154d75791e06d321895bb0f089481dc85384a85e658c6d8653</password_sha256_hex>
            <networks incl="networks" replace="replace">
                <ip>::/0</ip>
            </networks>
            <profile>readonly</profile>
            <quota>default</quota>
        </read>
    </users>
</yandex>
```

# 创建docker-compose.yml配置文件
```
cat > /data/docker/clickhouse/docker-compose.yml << EOF
version: '3.7'
services:
  clickhouse-server:
    image: yandex/clickhouse-server
    container_name: clickhouse-server
    ports:
      - 8123:8123
      - 9000:9000
    expose:
      - 8123
      - 9000
    volumes:
      - ./data:/var/lib/clickhouse
      - ./log:/var/log/clickhouse-server
      - ./conf/users.xml:/etc/clickhouse-server/users.xml
EOF
```

# 启动服务
```
cd /data/docker/clickhouse
docker-compose up -d
```

# 客户端使用
```
rpm --import https://repo.yandex.ru/clickhouse/CLICKHOUSE-KEY.GPG
yum-config-manager --add-repo https://repo.yandex.ru/clickhouse/rpm/stable/x86_64
yum -y install clickhouse-client
clickhouse-client -h127.0.0.1 --port=9000 -m -uroot --password Qgwp0VaF
```

# 基本语法

**创建数据库**
```
CREATE DATABASE IF NOT EXISTS channel_data_dev;
```
**创建普通表**
```
CREATE TABLE test(id UInt16,name String,contry String,create_date date ) ENGINE = MergeTree(create_date, (id), 8192);
```
**创建mysql映射库**
```
create database testdb ENGINE = MySQL('172.19.132.100:3306', 'testdb', 'root', 'runx@123');
```
**创建mysql映射表**
```
CREATE TABLE tb1
ENGINE = MergeTree
PARTITION BY toYYYYMM(pay_time)
ORDER BY pay_time AS
SELECT *
FROM mysql('127.0.0.1:3306', 'yayun', 'tb1', 'ch_repl', '123') ;
```
**创建单表物化视图**
```
create MATERIALIZED VIEW channel_data_dev_view.test1 ENGINE = MergeTree() PARTITION BY id ORDER BY id SETTINGS index_granularity = 8192 AS select * from testdb.test1;
````
**查看建表语句**
```
show create table test1;
```
