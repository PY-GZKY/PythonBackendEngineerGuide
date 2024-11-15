# 监控服务

- https://github.com/louislam/uptime-kuma
- https://github.com/louislam/uptime-kuma/wiki

## 如何使用 Upptime Kuma 搭建站点监控服务?

Uptime Kuma 是一款易于使用的自托管监控工具：

- 监控 HTTP（s） / TCP / HTTP（s） 关键字 / HTTP（s） 的正常运行时间 Json 查询 / Ping / DNS 记录 / 推送 / Steam 游戏服务器 / Docker 容器
- 花哨的、反应式的、快速的 UI/UX
- 通过 Telegram、Discord、Gotify、Slack、Pushover、电子邮件 （SMTP） 和 90+ 通知服务发送通知，单击此处查看完整列表
- 多语言，多个状态页面，证书信息


### 如何使用 Docker 启动 Upptime Kuma 服务?

```shell
docker run -d --restart=always -p 3001:3001 -v uptime-kuma:/app/data --name uptime-kuma louislam/uptime-kuma:1
```

> 启动后浏览到 http://localhost:3001 预览即可。

更新到最新的docker发布版本：

```shell
docker pull louislam/uptime-kuma:1
docker stop uptime-kuma
docker rm uptime-kuma

docker run -d --restart=always -p 3001:3001 -v uptime-kuma:/app/data --name uptime-kuma louislam/uptime-kuma:1
```

> 对于每个新版本，构建 docker 镜像都需要一些时间，如果还没有，请耐心等待。

---

## 如何使用 Prometheus + Grafana 进行可视化监控?

### 如何使用 Docker 启动 Prometheus 服务?

首先需要构造一个 `docker-compose.yml` 文件，内容如下：

```yaml
version: '3'

networks:
  monitoring:
    driver: bridge

volumes:
  prometheus_data: { }

services:
  grafana:
    image: grafana/grafana
    container_name: grafana
    restart: always
    ports:
      - "3000:3000"
    volumes:
      - ./grafana:/var/lib/grafana
    environment:
      - GF_DEFAULT_LOCALE=zh-CN

  mysql-exporter:
    image: prom/mysqld-exporter
    container_name: mysqld-exporter
    ports:
      - "9104:9104"
    volumes:
      - ./my.cnf:/.my.cnf
    networks:
      - monitoring
    depends_on:
      - prometheus

  node-exporter:
    image: prom/node-exporter:latest
    container_name: node-exporter
    restart: unless-stopped
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    command:
      - '--path.procfs=/host/proc'
      - '--path.rootfs=/rootfs'
      - '--path.sysfs=/host/sys'
      - '--collector.filesystem.mount-points-exclude=^/(sys|proc|dev|host|etc)($$|/)'
    ports:
      - "9100:9100"
    networks:
      - monitoring

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

这个docker-compose.yml文件中包含了三种服务：

- Grafana：用于展示监控数据
    - 在本地创建一个 `grafana` 目录，用于存储 Grafana 的数据
    - 通过 `GF_DEFAULT_LOCALE` 环境变量设置默认语言为中文
    - 通过 `ports` 指定端口映射到本地的 3000 端口
- Prometheus：用于存储监控数据
    - 在本地创建一个 `prometheus_data` 目录，用于存储 Prometheus 的数据
    - 通过 `volumes` 挂载 `prometheus.yml` 配置文件
    - 通过 `ports` 指定端口映射到本地的 9090 端口
- Exporter：用于收集监控数据
    - Node Exporter：用于收集主机的监控数据，如 CPU、内存、磁盘等，映射到本地的 9100 端口
    - MySQL Exporter：用于收集 MySQL 数据库的监控数据，如连接数、查询数、响应时间等，映射到本地的 9104 端口，my.cnf 是 MySQL 的配置文件

### 编写 prometheus.yml 配置文件

```yaml
global:
  scrape_interval: 5s

scrape_configs:
  - job_name: 'prometheus'
    scrape_interval: 5s
    static_configs:
      - targets: [ '1.1.1.1:9090' ]

  - job_name: 'node'
    static_configs:
      - targets: [ '1.1.1.1:9100' ]

  - job_name: 'mysql'
    static_configs:
      - targets: [ '1.1.1.1:9104' ]
```

这个配置文件中定义了三个 job：

- prometheus：用于监控 Prometheus 本身，也就是上面说到的启动的 Prometheus 服务到 9090 端口
- node：用于监控 Node Exporter，也就是上面说到的启动的 Node Exporter 服务到 9100 端口
- mysql：用于监控 MySQL Exporter，也就是上面说到的启动的 MySQL Exporter 服务到 9104 端口

值得注意的是，MySQL Exporter 需要一个 my.cnf 配置文件，内容如下：

```
[client]
host=1.1.1.1
port=30004
user=root
password=123456
```

最后，启动服务：

```shell
docker-compose up -d
```

> 启动后浏览到 http://localhost:3000，使用默认的用户名和密码 `admin` 登录，然后配置数据源和导入仪表盘。
