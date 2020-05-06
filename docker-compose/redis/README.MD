# 创建目录
```
mkdir /data/docker/redis/{data,conf}
```

# 创建redis配置文件
```
cat > /data/docker/redis/conf/redis.conf << EOF
requirepass AJA182ak!2D
#daemonize yes
bind 0.0.0.0
appendonly yes
EOF
```

# 创建docker-compose.yml配置文件
```
cat > /data/docker/redis/docker-compose.yml << EOF
version: '3.7'

services:
  redis:
    image: redis
    restart: always
    ports:
      - 6379:6379
    volumes:
      - ./conf/redis.conf:/etc/redis/redis.conf
      - ./data:/data
    command:
      redis-server /etc/redis/redis.conf
    networks:
      - custom_net
networks:
  custom_net:
    external:
      name: polaris_net
EOF
```

# 启动服务
```
cd /data/docker/redis
docker-compose up -d
```
