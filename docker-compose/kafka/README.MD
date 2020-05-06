# 创建目录
```
mkdir -p /data/docker/kafka/data/zookeeper/data
mkdir -p /data/docker/kafka/data/kafka
```

# 创建docker-compose配置文件
```
cat > /data/docker/kafka/docker-compose.yml << EOF
version: '3.7'
services:
  zookeeper:
    image: wurstmeister/zookeeper
    ports:
      - "2181:2181"
    volumes:
      - /data/docker/kafka/data/zookeeper/data:/opt/zookeeper-3.4.13/data
  kafka:
    image: wurstmeister/kafka
    volumes:
        - /etc/localtime:/etc/localtime
    ports:
      - "9092:9092"
    environment:
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:9092
      KAFKA_ADVERTISED_HOST_NAME: kafka
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_PORT: 9092
      KAFKA_LOG_RETENTION_HOURS: 120
      KAFKA_MESSAGE_MAX_BYTES: 10000000
      KAFKA_REPLICA_FETCH_MAX_BYTES: 10000000
      KAFKA_GROUP_MAX_SESSION_TIMEOUT_MS: 60000
      KAFKA_NUM_PARTITIONS: 3
      KAFKA_DELETE_RETENTION_MS: 1000
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /data/docker/kafka/data/kafka:/kafka
EOF
```

# 启动服务
```
cd /data/docker/kafka
docker-compose up -d
```
