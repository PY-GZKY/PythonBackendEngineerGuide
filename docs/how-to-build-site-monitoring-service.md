# 监控服务

- https://github.com/louislam/uptime-kuma
- https://github.com/louislam/uptime-kuma/wiki

## 如何安装并启动 uptime-kuma

Uptime Kuma 是一款易于使用的自托管监控工具。

### 特征：

- 监控 HTTP（s） / TCP / HTTP（s） 关键字 / HTTP（s） 的正常运行时间 Json 查询 / Ping / DNS 记录 / 推送 / Steam 游戏服务器 / Docker 容器
- 花哨的、反应式的、快速的 UI/UX
- 通过 Telegram、Discord、Gotify、Slack、Pushover、电子邮件 （SMTP） 和 90+ 通知服务发送通知，单击此处查看完整列表
- 多语言
- 多个状态页面
- Ping 图表
- 证书信息
- 代理支持
- 2FA 支持

使用 docker run 快速启动服务：

```shell
docker run -d --restart=always -p 3001:3001 -v uptime-kuma:/app/data --name uptime-kuma louislam/uptime-kuma:1
```

> 启动后浏览到 http://localhost:3001

更新到最新的docker发布版本：

```shell
docker pull louislam/uptime-kuma:1
docker stop uptime-kuma
docker rm uptime-kuma

# Default
docker run -d --restart=always -p 3001:3001 -v uptime-kuma:/app/data --name uptime-kuma louislam/uptime-kuma:1
```

对于每个新版本，构建 docker 镜像都需要一些时间，如果还没有，请耐心等待。

## 如何使用docker监控节点性能

一般来说采用 prometheus + grafana + exporter 的方式来监控各类服务的指标。

首先，使用 docker-compose.yml 文件启动一个p容器：

```yaml
version: '3'

networks:
  monitoring:
    driver: bridge

volumes:
  prometheus_data: {}

services:

  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    restart: unless-stopped
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - ./prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.console.libraries=/etc/prometheus/console_libraries'
      - '--web.console.templates=/etc/prometheus/consoles'
      - '--web.enable-lifecycle'
    ports:
      - "9090:9090"
    networks:
      - monitoring
```
