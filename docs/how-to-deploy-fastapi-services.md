# 如何快速部署 `FastAPI` 服务?

`fastapi` 官方推荐使用 `uvicron` 服务器来部署其服务。

`uvicorn` 是一款轻量快速的 `Python ASGI `框架(异步框架)，基于`uvloop`和`httptools`构建。

直到最近，`Python`还没有为`asyncio`框架提供最小的低级服务器/应用程序接口。 `ASGI`规范填补了这一空白，意味着我们现在能够开始构建一个可用于所有`asyncio`框架的通用工具集。

`ASGI`帮助实现一个`Python Web`框架生态系统，该框架在与IO绑定的上下文中实现高吞吐量方面与`Node`和`Go`竞争非常激烈。 它还提供对`HTTP/2`和`WebSockets`的支持，`WSGI`无法处理。

`Uvicorn`目前支持`HTTP/1.1`和`WebSockets`。 计划支持`HTTP/2`。

由于 `Uvicron` 的诞生，以及 `fastapi` 本身支持高性能的异步请求，在 `Python` 轻量级框架中有着质的飞跃。
这也是 `fastapi` 近来大火的原因，某些程度上更好的替代了 `flask` 的江湖地位。

### 安装 `Uvicron`

```shell
pip install uvicorn
```

## 如何构建一个简单的应用程序?

新建一个 main.py 文件并编写代码：

```python
import uvicorn
from fastapi import FastAPI

app = FastAPI()


@app.get("/")
def home(name="fastapi"):
    return f"Hello! {name}"


if __name__ == "__main__":
    uvicorn.run(app, host='127.0.0.1', port=8000, debug=True)
```

可以看到 `fastapi` 的语法和 `flask`非常接近，不管是从设计模式还是感官上来说都有着异曲同工之妙。

运行 `main.py` 文件：

````shell
python main.py
````

这样一个简单的`fastapi` 应用就起来了并且使用了 `uvicron` 服务器启动
输出如下:

```shell
INFO: Uvicorn running on http://0.0.0.0:8000 (Press CTRL+C to quit)
INFO: Started reloader process [37928] using statreload
INFO: Started server process [41144]
INFO: Waiting for application startup.
2021-04-08 10:53:29.938 | DEBUG    | app.api:connect_to_mysql:73 - MYSQL 数据库初始化成功 ... DONE
2021-04-08 10:53:30.101 | DEBUG    | app.api:connect_to_mongo:89 - MONGODB 数据库初始化成功 ... DONE
2021-04-08 10:53:30.101 | DEBUG    | app.api:connect_to_ws:119 - WEBSOCKET 连接初始化成功 ... DONE
INFO: Application startup complete.
```

在开发过程中，我们想知道代码被改变时接口或页面上实时的同步更改，指定 `--reload `参数为 `True` 可以解决这个问题，也叫热加载。

```shell
if __name__ == "__main__":
    uvicorn.run(app='main:app', host="0.0.0.0", port=8000,reload=True, debug=True)
```

## 如何启动 `uvicorn`?

在线上部署的时候，我们需要将 `web服务`永久的在后台运行，可以使用:

```shell
nohup uvicorn main:app --workers 4 --host=0.0.0.0 --port 8080 >> ./output.log 2>&1 &
```

`nohup` 是 `linux` 操作系统中用于运行后台常驻服务的关键字，我们将输出`日志`到 `./output.log` 文件，方便查看。

我们启用了` 4` 个线程，一般来说线程数为宿主机内核的双倍数，比如 `2` 核的主机即开启 `4` 个线程即可。

查看 uvicorn 是否正常工作

```shell
ps -ef | grep 8080
```

```shell
root  3180    1   0 4月03 ?        00:00:00 /root/anaconda3/bin/python /root/anaconda3/bin/uvicorn main:app --workers 4 --host=0.0.0.0 --port 8080
root  3855  6709  0 11:27 pts/1    00:00:00 grep --color=auto 8006
```

`--port` 参数对外开放了 `8080` 端口，通过 `http://IP:8080` 访问目标主机可以看到服务已经正常启。

`nohup` 会将所有的 `print`、 `log` 、`debug` 输出的信息汇总到`./output.log` 文件，大概是这样的:

```bash
2021-04-08 09:00:06.622 | DEBUG    | app.api.db.mysqlDB:bulk_save:21 - [+] 插入成功 !!! =========  erp3c_customer_delivery   ========
2021-04-08 09:00:06.639 | DEBUG    | app.api.db.mysqlDB:bulk_save:21 - [+] 插入成功 !!! =========  erp3c_customer_delivery   ========
2021-04-08 09:00:07.198 | DEBUG    | app.api.api_v1.spider_m.erp3c_delivery_out.erp_delivery_16h:crawl:252 - [+]  更新成功 ..... done
{'status_code': 200, 'message': '获取状态成功', 'data': 1}
INFO:     113.66.254.2:54315 - "POST /api/v1/erp3c_delivery_16h/1f11b32dec39262e84df HTTP/1.1" 200 OK
```

## 如何使用 `Uvicorn` + `Nginx` 部署应用?

如果是`单机/单服务器`应用的话，通过上面的部署之后项目已经完美启动。

但是我们总是热衷于使用 `Nginx` 来反向代理我们的`web 服务`，`Nginx` 似乎能够让我们的网站更像一个网站。

更重要的方面是 `Nginx` 确实是一款高效实用的`反向代理服务器`，安装简单，配置容易，支持负载均衡，确为线上部署的一把利器，受到广泛开发者的青睐也在情理之中了。

> 接下来我们要构建  `Uvicorn` + `Nginx` 这套组合，并让这套组合拳有效的运行于我们的服务器中。

`Nginx` 的可配置项非常多，我们这里不做过多的详述，毕竟篇幅不小。

这里假设已经成功搭建并启动了 `Nginx` 反向代理服务器，如果未启动则通过文档中的命令重启即可。

查看 `Nginx` 安装的所在目录：

```shell
[root@~] whereis nginx
nginx: /usr/local/nginx
```

进入` /usr/local/nginx/conf` 并编辑 `nginx.conf`配置文件

```shell
upstream app {
  ip_hash;
  server localhost:8080;
}

server {
  listen 80;
  server_name localhost;
  
  location /static/ {
    autoindex on;
    alias /code/collected_static/;
  }
  
  location / {
    proxy_pass http://app/;
  }
}
```

这样 `Nginx` 将监听 `:8080 `端口服务并转发到宿主机的` 80` 端口上，如果是`前后端分离`项目则不做 `location /static/` 配置，只提供 `api` 接口服务即可。

否则则要声明项目的静态文件目录，比如你使用的是 Vue 编写的前端项目，那么需要将打包好的前端项目放于 `location /static/` 之内，通过直接访问 `Vue`页面，再由`Vue`调用我们的后端接口

验证 `Nginx` 配置文件是否异常:

```shell
[root@gzky_gz conf] /usr/local/nginx/sbin/nginx  -t
nginx: the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /usr/local/nginx/conf/nginx.conf test is successful
```

出现以上信息则说明配置文件无误，否则根据报错行数修正配置文件即可。

> 只要确保 `:8080` 服务正常和`Nginx 配置`文件无异常，项目将顺利启动。

通过 `http://IP `地址访问项目根路径，打开控制台发现由 `Nginx` 提供服务。

![](images/Nginx.png)

需要注意的是如果我们使用 `https://IP ` 访问项目会出现 `无法访问此网站` 的情况，原因是我们并没有做 https 配置和重定向，这需要一些 ssl 证书的支持，这个不做详述。

## 如何使用 `Docker` 进行部署?

> `Docker` 是一个开源的应用容器引擎，基于 `Go` 语言 并遵从 `Apache2.0 `协议开源。
> `Docker` 可以让开发者打包他们的应用以及依赖包到一个轻量级、可移植的容器中，然后发布到任何流行的 `Linux` 机器上，也可以实现虚拟化。
> 号称`远程小型虚拟机`，但是更轻更快 ！

当你应对不同的语言版本、不同的框架版本而举足无措的时候，`docker` 可能成为你的第一选择。

假设我们的工程目录 `code`，请在 `code` 下:

- 生成 `requirements.txt` 文件

```shell
pip freeze > requirements.txt
```

- `Nginx` 配置文件

在 `code/config/nginx` 下新建 `web_app.conf` 文件并写入以下配置

```shell
upstream app {
  ip_hash;
  server localhost:8080;
}

server {
  listen 80;
  server_name localhost;
  
  location /static/ {
    autoindex on;
    alias /code/collected_static/;
  }
  
  location / {
    proxy_pass http://app/;
  }
}
```

- 构建 `Dockerfile` 文件

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

# 新建和指定工程目录为 code
RUN mkdir /code
WORKDIR /code
# 升级 pip 和安装依赖
RUN pip install pip -U -i https://pypi.tuna.tsinghua.edu.cn/simple
ADD requirements.txt /code/
RUN pip install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple
ADD . /code/
```

`docker` 本质上是`Linux`操作系统上的一个进程，但又是一个独立的环境，容器间相互隔离，同样具有 `Unix` 内核。

### 使用 `Docker-compose`

既然我们已经在本地安装了 `Python`版本，可以使用 `pip` 安装 `Docker-compose`

```shell
pip install Docker-compose
```

安装后即刻生效，验证是否正常工作:

```shell
[root@gzky_gz conf]# Docker-compose -v
Docker-compose version 1.28.6
```

### 构建 `docker-compose.yml` 文件

```yml
version: "3"
services:
  app:
    restart: always
    build: .
    # 启动命令
    command: bash -c "nohup uvicorn main:app --workers 4 --host=0.0.0.0 --port 8080 >> ./output.log 2>&1 &"
    # 挂载 code 目录
    volumes:
      - .:/code
    # expose 只对内部暴露 8080
    expose:
      - "8080"
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
Docker-compose up
```

此命令会顺序执行:

- 编译 `Dockerfile` 文件，会在`docker`安装 `Python3.8` 镜像 以及`requirements.txt`中的所有库
- 安装 `Nginx` 最新版本并同步挂载`./config/nginx` 目录到 容器中的 `/etc/nginx/conf.d` 目录，也就是修改其配置文件了
- 启动`web服务`暴露 `:8080`，这里没使用`ports` 端口映射是因为我们只是把`8080`端口转发给 `Nginx` 而已，而不是对外开放
- 启动 `Nginx`，注意是要对外暴露 `80` 端口，所以使用 `ports` 关键字

容器启动成功后，通过 `http://IP` 地址访问项目，实际效果和本地宿主机部署一样。

## 如何集成 `Vue` 项目?

用官方的话说，`Vue` 是一套用于构建用户界面的渐进式框架。 与其它大型框架不同的是，`Vue` 被设计为可以自底向上逐层应用。

这里我们主要注重基于 `Fastapi` + `Vue` 的前后端分离的部署部分。

如果你没有做一些额外的特殊的设置的话：

```shell
npm run build
```

通过以上命令可以对 `Vue` 项目进行打包，不出意外的话就生成一个默认名为 `dist` 的静态文件目录，也就是最后打包的结果了。

至于`fastapi`的后端接口则可以存在于任何一台有`公网ip`的机器。

## 如何使用 `Nginx`部署`Vue`项目?

在 `nginx.conf`中编辑以下配置

```shell
server {
  listen 80;
  server_name localhost;
  
  location / {
            root   /code/dist;
            index  index.html index.html;
        }
}
```

让 `Nginx` 找到 `/code/dist` 目录并将目录下的`index.html` 作为`Nginx`首页，`纯静态部署`总是如此的简单。

## 如何部署多个`Vue`项目?

虽然一个`Nginx`也能够部署多个 `Vue`项目，但是我还是不建议这样的做法，一旦`Vue`项目体量过大，调度的接口过多，则会发生性能不足的情况。
可以使用搭建多个`Nginx` 服务器来挂载多个`Vue`项目(或其他前端项目)，一个`Nginx`对应一个`Vue`项目。
而理论上，只要我们的机器的`cpu`和`内存`足够，可以搭建很多很多的`Nginx`，只需改变其默认端口即可。

## 如何使用 acme.sh 部署 SSL 证书?

### 安装 acme.sh

安装很简单，就一个命令：

```bash
curl https://get.acme.sh | sh
```

普通用户和 root 用户都可以安装使用。这会安装在 ~/.acme.sh/ 目录下，以后生成的证书也会在这里面，按照域名为文件夹安置。

理论上会自动添加一个 acme.sh 别名，但有时候并不会生成，需要手动执行以下命令： `source ~/.bashrc`

### 使用 dns api 的模式进行证书申请

获取 AccessKey ID和AccessKey Secret

```bash
export Ali_Key="key"
export Ali_Secret="key Secret"

# 下次就不用再次执行这个命令了
./acme.sh --issue --dns dns_ali -d *.example.com --force
```

### 生成 .key 和 .crt证书

```shell
./acme.sh --install-cert -d "*.gzsonic.com" --cert-file cert.crt --key-file key.key --fullchain-file ca.crt
```

.key 和 .crt证书可用于 nginx 证书部署.

> 参考: https://github.com/acmesh-official/acme.sh

### 自动更新证书

Let's 的证书有效期为60天

目前手动添加DNS获取证书的方式无法自动更新，但是使用DNS API的方式进行获取证书可以在 60 天以后会自动更新, 你无需任何操作.

强制执行更新任务: `acme.sh --cron -f`

### 升级

目前由于 acme 协议和 Let`s CA 都在频繁的更新, 因此 acme.sh 也经常更新以保持同步.

升级 acme.sh 到最新版 : `acme.sh --upgrade`

如果你不想手动升级, 可以开启自动升级: `acme.sh --upgrade --auto-upgrade` 之后, acme.sh 就会自动保持更新了.

你也可以随时关闭自动更新: `acme.sh --upgrade --auto-upgrade 0`







