# 如何使用 `Docker` 部署

Docker 的优点：

- 灵活，不管你的应用环境有多复杂，都可以打包成一个镜像
- 便携，只需要打包一次，就可以部署到任何地方运行
- 高效，与宿主机共享操作系统内核，效率很高
- 安全，不同的容器之间互不干扰，崩了不影响其他应用

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

## Portainer 可视化管理

Portainer 是一个轻量级的容器管理工具，它提供了一个直观的 web 界面，使得用户能够方便地管理 Docker 环境。通过
Portainer，用户可以轻松查看和管理容器、镜像、网络和数据卷等Docker资源。它支持创建、删除、启动和停止容器，以及创建、删除和查看镜像、网络和数据卷。此外，Portainer
允许用户查看容器日志、详细信息、资源使用情况、环境变量、端口映射、文件系统、进程、事件和健康状态，从而提供了全面的容器监控和管理功能。Portainer
的优势在于它的用户友好界面和强大的功能集，使得Docker管理变得更加简单和高效，适合各种规模的环境和不同技能水平的用户。

### 安装

```shell
docker run -d -p 9000:9000 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v /data/portainer:/data portainer/portainer
```

然后访问地址: http://localhost:9000 进行用户注册和登录(初始化)。

![Portainer](./images/portainer_demo.png)

## Harbor 访问控制

什么是Harbor？

Harbor 是一个开源注册表，它通过策略和基于角色的访问控制来保护工件，确保扫描映像且没有漏洞，并将映像标记为受信任。Harbor 是
CNCF 毕业项目，可提供合规性、性能和互操作性，帮助您跨 Kubernetes 和 Docker 等云原生计算平台一致、安全地管理工件。