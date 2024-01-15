# 如何使用 `Docker` 部署

## 打包镜像

### 1. build Dockerfile 编译镜像

```shell
docker build -t server -f Dockerfile.train .
```

### 2. save image 打包保存镜像

```shell
docker save train_server_offline_app:latest | gzip > app.tar.gz
docker save train_server_offline_train:latest | gzip > train.tar.gz
docker save mysql:5.7.30 | gzip > mysql.tar.gz
docker save mongo:4.4.6 | gzip > mongo.tar.gz
docker save nginx:latest | gzip > nginx.tar.gz
```

## Redis

```shell
docker run -d --name redis \
--restart=always \
-v /data/wutong/redis/conf/redis.conf:/etc/redis.conf \
-v /data/wutong/redis/data:/data \
-p 6379:6379 \
redis:5.0.3 redis-server /etc/redis.conf
```


### 清除所有异常退出的容器

```shell
docker ps -a | grep Exited | awk '{print $1}' | xargs docker rm
```

### 清空所有镜像名称为空的镜像

```shell
docker images -q --filter "dangling=true" | xargs docker rmi
```

