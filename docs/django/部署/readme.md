
> 开始部署 `django` 服务


![Python](https://img.shields.io/badge/Python-3.8+-blue)
![Django](https://img.shields.io/badge/Django-3.0-brightgreen)
![Nginx](https://img.shields.io/badge/Nginx-1.16.1-red)
![Mysql](https://img.shields.io/badge/Mysql-5.7-blue)

## `Uwsgi`
> `Django` 官方推荐使用 `uwsgi` 服务器部署 `django` 应用

`uWSGI` 是一个快速的、纯C语言开发的、自维护的、对开发者友好的 WSGI 服务器， 旨在提供专业的 `Python web`应用发布和开发。
可使用 `C/C++/Objective-C` 来为 `uWSGI` 编写插件。

### 安装 `Uwsgi`
```shell
pip install uwsgi
```
验证是否安装成功:
```shell
[root@gzky_gz conf] uwsgi --version
2.0.19.1
```
> 没有问题 ！！
> 
## 构建一个简单的 `Django` 应用

创建一个 Django 项目
```shell
django-admin startproject HelloWorld
```
查看项目结构
```shell
$ cd HelloWorld/
$ tree
.
|-- HelloWorld
|   |-- example.py
|   |-- asgi.py
|   |-- settings.py
|   |-- urls.py
|   `-- wsgi.py
`-- manage.py
```

可以看到 `django3.0` 除了 `wsgi.py` 之外还自动创建了 `asgi.py`，这是因为`Django` 计划在`3.0` 版本引入异步请求
- `__init__` 初始化文件
- `asgi.py` 使用异步服务器启动项目
- `wsgi.py` 使用非异步服务器启动项目
- `settings.py` 主配置文件入口
- `urls.py` 主路由分发文件
- `manage.py` 主启动文件

`django` 提供默认`首页` ！！

运行一下命令启动项目:
```shell
python manage.py runserver 0.0.0.0:8000
```

访问 `localhost:8000` 小火箭直接起飞 ！！


## `Uwsgi` + `Django` 

接下来使用 `Uwsgi` 服务器启动我们刚刚创建的 `django` 项目(虽然只有一个默认首页)
在 `HelloWorld` 目录下新建 `script` 目录用于存放 `Uwsgi` 相关配置以及日志文件。

查看目录结构
```shell
$ cd script/
$ tree
.
|-- script
|   |-- uwsgi.ini
|   |-- uwsgi.pid
|   |-- uwsgi.log
```

- `uwsgi.ini` 主配置文件
- `uwsgi.pid` 存放进程`pid`，用于停止，重启等操作
- `uwsgi.log`日志文件

### `uwsgi.ini`
```shell
[uwsgi]

# 项目目录
chdir = /HelloWorld

# 启动uwsgi的用户名和用户组
uid = root
gid = root

# 指定项目的application
module = djangoBlog.wsgi:application

# 启用主进程
master = true

# 进程个数
workers = 4
pidfile = ./uwsgi.pid

# 自动移除unix Socket和pid文件当服务停止的时候
vacuum = true

# 序列化接受的内容，如果可能的话
thunder-lock = true

# 启用线程
enable-threads = true

# 设置自中断时间
harakiri = 600
uwsgi_read_timeout = 600

# 设置缓冲
post-buffering = 4096

# 设置日志目录
daemonize = ./uwsgi.log
```

### `uwsgi` 开启 `django`服务

```shell
uwsgi --ini uwsgi.ini
```

查看进程，发现启动成功:
```shell
[root@gzky_gz conf]# ps -ef | grep uwsgi
root      6998 25585  0 4月01 ?       00:00:02 /root/miniconda3/bin/uwsgi --ini uwsgi.ini
root      6999 25585  0 4月01 ?       00:00:02 /root/miniconda3/bin/uwsgi --ini uwsgi.ini
root      7000 25585  0 4月01 ?       00:00:02 /root/miniconda3/bin/uwsgi --ini uwsgi.ini
root      7001 25585  0 4月01 ?       00:00:02 /root/miniconda3/bin/uwsgi --ini uwsgi.ini
```

访问 `localhost:8000` 小火箭又起飞 ！！

至于 `uwsgi` 日志大概是这样的:
```shell
WSGI app 0 (mountpoint='') ready in 0 seconds on interpreter 0x19fc590 pid: 25585 (default app)
*** WARNING: you are running uWSGI as root !!! (use the --uid flag) *** 
*** uWSGI is running in multiple interpreter mode ***
gracefully (RE)spawned uWSGI master process (pid: 25585)
spawned uWSGI worker 1 (pid: 6998, cores: 1)
spawned uWSGI worker 2 (pid: 6999, cores: 1)
spawned uWSGI worker 3 (pid: 7000, cores: 1)
spawned uWSGI worker 4 (pid: 7001, cores: 1)
spawned uWSGI worker 5 (pid: 7002, cores: 1)
[pid: 6998|app: 0|req: 1/1] 116.22.198.79 () {44 vars in 1139 bytes} [Thu Apr  1 09:40:58 2021] GET /admin/ => generated 21803 bytes in 190 msecs (HTTP/1.1 200) 7 headers in 373 bytes (1 switches on core 0)
[pid: 6999|app: 0|req: 1/2] 116.22.198.79 () {44 vars in 1182 bytes} [Thu Apr  1 09:40:59 2021] GET /admin/article/article/ => generated 43275 bytes in 211 msecs (HTTP/1.1 200) 7 headers in 396 bytes (2 switches on core 0)
[pid: 7000|app: 0|req: 1/3] 116.22.198.79 () {42 vars in 1063 bytes} [Thu Apr  1 09:40:59 2021] GET /admin/jsi18n/ => generated 7722 bytes in 98 msecs (HTTP/1.1 200) 4 headers in 132 bytes (1 switches on core 0)
[pid: 7001|app: 0|req: 1/4] 116.22.198.79 () {42 vars in 1072 bytes} [Thu Apr  1 09:40:59 2021] GET /media/face/2.jpg => generated 332 bytes in 149 msecs (HTTP/1.1 404) 3 headers in 116 bytes (1 switches on core 0)
[pid: 7002|app: 0|req: 1/5] 116.22.198.79 () {46 vars in 1099 bytes} [Thu Apr  1 09:40:59 2021] GET /favicon.ico => generated 0 bytes in 82 msecs (HTTP/1.1 302) 4 headers in 140 bytes (1 switches on core 0)
[pid: 6998|app: 0|req: 2/6] 116.22.198.79 () {44 vars in 1216 bytes} [Thu Apr  1 09:41:00 2021] GET /admin/article/article/4/change/ => generated 37389 bytes in 52 msecs (HTTP/1.1 200) 7 headers in 396 bytes (2 switches on core 0)
[pid: 6999|app: 0|req: 2/7] 116.22.198.79 () {42 vars in 1072 bytes} [Thu Apr  1 09:41:00 2021] GET /admin/jsi18n/ => generated 7722 bytes in 6 msecs (HTTP/1.1 200) 4 headers in 132 bytes (1 switches on core 0)
[pid: 7000|app: 0|req: 2/8] 116.22.198.79 () {42 vars in 1081 bytes} [Thu Apr  1 09:41:01 2021] GET /media/face/2.jpg => generated 332 bytes in 80 msecs (HTTP/1.1 404) 3 headers in 116 bytes (1 switches on core 0)
[pid: 7001|app: 0|req: 2/9] 116.22.198.79 () {52 vars in 1409 bytes} [Thu Apr  1 09:41:07 2021] POST /admin/article/article/4/change/ => generated 0 bytes in 46 msecs (HTTP/1.1 302) 8 headers in 603 bytes (1 switches on core 0)
[pid: 7002|app: 0|req: 2/10] 116.22.198.79 () {46 vars in 1534 bytes} [Thu Apr  1 09:41:08 2021] GET /admin/article/article/ => generated 43575 bytes in 126 msecs (HTTP/1.1 200) 8 headers in 480 bytes (2 switches on core 0)
[pid: 6998|app: 0|req: 3/11] 116.22.198.79 () {42 vars in 1063 bytes} [Thu Apr  1 09:41:08 2021] GET /admin/jsi18n/ => generated 7722 bytes in 6 msecs (HTTP/1.1 200) 4 headers in 132 bytes (1 switches on core 0)
[pid: 6999|app: 0|req: 3/12] 116.22.198.79 () {42 vars in 1072 bytes} [Thu Apr  1 09:41:08 2021] GET /media/face/2.jpg => generated 332 bytes in 10 msecs (HTTP/1.1 404) 3 headers in 116 bytes (1 switches on core 0)
[pid: 7000|app: 0|req: 3/13] 116.22.198.79 () {44 vars in 1139 bytes} [Thu Apr  1 09:41:13 2021] GET /admin/ => generated 21831 bytes in 66 msecs (HTTP/1.1 200) 7 headers in 373 bytes (1 switches on core 0)
[pid: 7001|app: 0|req: 3/14] 116.22.198.79 () {44 vars in 1182 bytes} [Thu Apr  1 09:41:13 2021] GET /admin/article/article/ => generated 43370 bytes in 72 msecs (HTTP/1.1 200) 7 headers in 396 bytes (2 switches on core 0)
[pid: 7002|app: 0|req: 3/15] 116.22.198.79 () {42 vars in 1063 bytes} [Thu Apr  1 09:41:14 2021] GET /admin/jsi18n/ => generated 7722 bytes in 6 msecs (HTTP/1.1 200) 4 headers in 132 bytes (1 switches on core 0)
[pid: 6998|app: 0|req: 4/16] 116.22.198.79 () {42 vars in 1072 bytes} [Thu Apr  1 09:41:14 2021] GET /media/face/2.jpg => generated 332 bytes in 9 msecs (HTTP/1.1 404) 3 headers in 116 bytes (1 switches on core 0)
[pid: 6999|app: 0|req: 4/17] 116.22.198.79 () {44 vars in 1216 bytes} [Thu Apr  1 09:41:15 2021] GET /admin/article/article/2/change/ => generated 37885 bytes in 81 msecs (HTTP/1.1 200) 7 headers in 396 bytes (2 switches on core 0)
[pid: 7000|app: 0|req: 4/18] 116.22.198.79 () {42 vars in 1072 bytes} [Thu Apr  1 09:41:15 2021] GET /admin/jsi18n/ => generated 7722 bytes in 5 msecs (HTTP/1.1 200) 4 headers in 132 bytes (1 switches on core 0)
[pid: 7001|app: 0|req: 4/19] 116.22.198.79 () {42 vars in 1081 bytes} [Thu Apr  1 09:41:15 2021] GET /media/face/2.jpg => generated 332 bytes in 10 msecs (HTTP/1.1 404) 3 headers in 116 bytes (1 switches on core 0)
[pid: 7002|app: 0|req: 4/20] 116.22.198.79 () {46 vars in 1099 bytes} [Thu Apr  1 09:41:16 2021] GET /favicon.ico => generated 0 bytes in 0 msecs (HTTP/1.1 302) 4 headers in 140 bytes (1 switches on core 0)
[pid: 6998|app: 0|req: 5/21] 116.22.198.79 () {52 vars in 1409 bytes} [Thu Apr  1 09:41:21 2021] POST /admin/article/article/2/change/ => generated 0 bytes in 29 msecs (HTTP/1.1 302) 8 headers in 561 bytes (1 switches on core 0)
[pid: 6999|app: 0|req: 5/22] 116.22.198.79 () {46 vars in 1492 bytes} [Thu Apr  1 09:41:21 2021] GET /admin/article/article/ => generated 43630 bytes in 44 msecs (HTTP/1.1 200) 8 headers in 480 bytes (2 switches on core 0)
[pid: 7000|app: 0|req: 5/23] 116.22.198.79 () {42 vars in 1063 bytes} [Thu Apr  1 09:41:21 2021] GET /admin/jsi18n/ => generated 7722 bytes in 5 msecs (HTTP/1.1 200) 4 headers in 132 bytes (1 switches on core 0)
[pid: 7001|app: 0|req: 5/24] 116.22.198.79 () {44 vars in 1094 bytes} [Thu Apr  1 09:41:25 2021] GET / => generated 22244 bytes in 87 msecs (HTTP/1.1 200) 3 headers in 111 bytes (1 switches on core 0)
[pid: 7002|app: 0|req: 5/25] 116.22.198.79 () {46 vars in 1093 bytes} [Thu Apr  1 09:41:26 2021] GET /favicon.ico => generated 0 bytes in 0 msecs (HTTP/1.1 302) 4 headers in 140 bytes (1 switches on core 0)
[pid: 6998|app: 0|req: 6/26] 116.22.198.79 () {44 vars in 1094 bytes} [Thu Apr  1 09:41:31 2021] GET / => generated 22244 bytes in 82 msecs (HTTP/1.1 200) 3 headers in 111 bytes (1 switches on core 0)
[pid: 6999|app: 0|req: 6/27] 116.22.198.79 () {46 vars in 1093 bytes} [Thu Apr  1 09:41:32 2021] GET /favicon.ico => generated 0 bytes in 0 msecs (HTTP/1.1 302) 4 headers in 140 bytes (1 switches on core 0)
[pid: 7000|app: 0|req: 6/28] 116.22.198.79 () {44 vars in 1111 bytes} [Thu Apr  1 09:41:46 2021] GET /about/ => generated 30944 bytes in 27 msecs (HTTP/1.1 200) 3 headers in 111 bytes (2 switches on core 0)
```

>至此，`Uwsgi` 部署 `Django` 完毕 ！！

## `Uwsgi` + `Nginx`

如果是`单机/单服务器`应用的话，通过上面的部署之后项目已经完美启动。
但是人们总是热衷于使用 `Nginx` 来反向代理我们的`web 服务`，`Nginx` 似乎能够让我们的网站更像一个网站 ！！

更重要的方面是 `Nginx` 确实是一款高效实用的反向代理服务器，安装简单，配置容易，支持负载均衡，确为线上部署的一把利器，受到广泛开发者的青睐也在情理之中了。

> 接下来我们要构建  `Uwsgi` + `Nginx` 这套组合，并让这套组合拳有效的运行于我们的服务器中 ！！

`Nginx` 的可配置项非常多，我们这里不做过多的详述，毕竟篇幅不小。

通过 [安装依赖](initialization/installer.md) 这一章，我们已经成功搭建并启动了 Nginx 反向代理服务器，如果未启动则通过文档中的命令重启即可！！


查看 `Nginx` 安装的所在目录
```shell
[root@gzky_gz ~] whereis nginx
nginx: /usr/local/nginx
```

进入` /usr/local/nginx/conf` 并编辑 `nginx.conf`配置文件

```shell
upstream app {
  ip_hash;
  server localhost:8000;
}

server {
  listen 80;
  server_name localhost;
  
  location / {
    proxy_pass http://app/;
  }
}
```

退出并保存 ！！

这样 `Nginx` 将监听 `:8000 `端口服务并转发到宿主机的` 80` 端口上，如果是`前后端分离`项目则不做 `location /static/` 配置，只提供 `api` 接口服务即可。

否则则要声明项目的静态文件目录，比如你使用的是 `Vue` 编写的前端项目，那么需要将打包好的前端项目放于 `location /static/` 之内，通过直接访问 `Vue`页面，再由`Vue`调用我们的后端接口

验证 `Nginx` 配置文件是否异常:
```shell
[root@gzky_gz conf] /usr/local/nginx/sbin/nginx  -t
nginx: the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /usr/local/nginx/conf/nginx.conf test is successful
```
出现以上信息则说明配置文件无误，否则根据报错行数修正配置文件即可 ！！

> 只要确保 `:8000` 服务正常和`Nginx 配置`文件无异常，项目将顺利启动 ！！


通过 `http://IP `地址访问项目根路径，打开控制台发现由 `Nginx` 提供服务

![](../../fastapi/部署/images/Nginx.png)


需要注意的是如果我们使用 `https://IP ` 访问项目会出现 `无法访问此网站` 的情况，原因是我们并没有做 https 配置和重定向，这需要一些 `ssl` 证书的支持，这个不做详述。

> 至此，一个简单的 `Uwsgi` +` Nginx` 组合部署完毕 ！！

## `Uwsgi` + `Nginx` + `Docker`

> `Docker` 是一个开源的应用容器引擎，基于 `Go` 语言 并遵从 `Apache2.0 `协议开源。 
> `Docker` 可以让开发者打包他们的应用以及依赖包到一个轻量级、可移植的容器中，然后发布到任何流行的 `Linux` 机器上，也可以实现虚拟化。
> 号称`远程小型虚拟机`，但是更轻更快 ！！

当你应对不同的语言版本、不同的框架版本而举足无措的时候，`docker` 可能成为你的第一选择 ！！

请在 `HelloWorld` 目录下:

### 生成 `requirements.txt` 文件
```shell
pip freeze > requirements.txt
```

### `Nginx` 配置文件
在 `HelloWorld/config/nginx` 下新建 `web_app.conf` 文件并写入以下配置
```shell
upstream app {
  ip_hash;
  server localhost:8000;
}

server {
  listen 80;
  server_name localhost;
  
  location / {
    proxy_pass http://app/;
  }
}
```

### 构建 `Dockerfile` 文件

```shell
# 基于 Py3.8 构建
FROM python:3.8
ENV PYTHONUNBUFFERED 1

# 换源
RUN echo \
deb https://mirrors.tuna.tsinghua.edu.cn/debian/ buster main contrib non-free\
deb https://mirrors.tuna.tsinghua.edu.cn/debian/ buster-updates main contrib non-free\
deb https://mirrors.tuna.tsinghua.edu.cn/debian/ buster-backports main contrib non-free\
deb https://mirrors.tuna.tsinghua.edu.cn/debian-security buster/updates main contrib non-free\
    > /etc/apt/sources.list

# 更新源
RUN apt-get update
# 安装必要的依赖项
RUN apt-get install python3-dev default-libmysqlclient-dev -y
# 新建和指定工程目录为 HelloWorld
RUN mkdir /HelloWorld
WORKDIR /HelloWorld
# 升级 pip 和安装依赖
RUN pip install pip -U -i https://pypi.tuna.tsinghua.edu.cn/simple
ADD requirements.txt /HelloWorld/
RUN pip install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple
ADD . /HelloWorld/
```

`docker` 本质上是`Linux`操作系统上的一个进程，但又是一个独立的环境，容器间相互隔离，同样具有 `Unix` 内核。

### 安装 `Docker-compose`

既然我们已经在本地安装了 `Python`版本，可以使用 `pip` 安装 `Docker-compose`
```shell
pip install docker-compose
```

> 安装后即刻生效

验证是否正常工作:
```shell
[root@gzky_gz conf]# docker-compose -v
docker-compose version 1.28.6
```
> 没有问题 ！！


### 构建 `docker-compose.yml` 文件

```yml
version: "3"
services:
  app:
    restart: always
    build: .
    # 启动命令
    command: bash -c "uwsgi --ini ./script/uwsgi.ini"
    # 挂载 code 目录
    volumes:
      - .:/HelloWorld
    # expose 只对内部暴露 8080
    expose:
      - "8000"
    networks:
      - web_network
  nginx:
    restart: always
    image: nginx:latest
    ports:
      - "80:80"
    volumes:
      - ./config/nginx:/etc/nginx/conf.d
    depends_on:
      - app
    networks:
      - web_network
networks:
  web_network:
    driver: bridge
```

### 开始 `build` 并启动容器

```shell
# 先不要 -d 后台启动，看看build详细情况
docker-compose up
```

此命令会顺序执行:
- 编译 `Dockerfile` 文件，会在`docker`安装 `Python3.8` 镜像 以及`requirements.txt`中的所有库
- 安装 `Nginx` 最新版本并同步挂载`./config/nginx` 目录到 容器中的 `/etc/nginx/conf.d` 目录，也就是修改其配置文件了
- 启动 `web服务`暴露 `:8000`，这里没使用 `ports` 端口映射是因为我们只是把`8000`端口转发给 `Nginx` 而已，而不是对外开放
- 启动 `Nginx`，注意是要对外暴露 `80` 端口，所以使用 `ports` 关键字


容器启动成功后，通过 `http://IP` 地址访问项目，实际效果和本地宿主机部署一样。

> 至此，`Uwsgi` + `Nginx` + `Docker` 部署成功 !!

## `Gunicorn`

除了 `Uwsgi`，很多人还喜欢用 Gunicorn 或 Uvicorn 来部署应用，我们可以自由搭配使用。

根据一些比较全面的测试统计(个人并未实际测试过)，`Uwsgi` 性能要比 Gunicorn web 服务器高出许多，这也是很多开发者
选择Uwsgi 来作为django 常用服务器的原因，当然也有不少人使用 Gunicorn ！！


### 安装 `Gunicorn`

```shell
pip install Gunicorn
```

### `Gunicorn` 配置文件
```shell

```


### 启动 `Gunicorn`

```shell
gunicorn HelloWorld.wsgi:application --bind 0.0.0.0:8000
```
或通过配置文件启动

```shell

```

> `Uvicorn` 部署可参照 [uvicorn部署FastAPI](fastapi/deploy/fastapi.md)


