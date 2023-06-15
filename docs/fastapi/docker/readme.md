### requirements.txt

在进入项目根目录之后，我们需要添加项目的 requirements.txt 文件，这个文件是用来指定我们项目的依赖的。大抵如此：

```txt
addict~=2.4.0
aioredis~=1.3.1
APScheduler~=3.9.1
emails~=0.6
fastapi~=0.66.0
fastapi_cache~=0.1.0
fastapi-cache2
fastapi_profiler~=1.0.0
gevent_socketio~=0.3.6
GitPython~=3.1.29
jose~=1.0.0
ldap3~=2.9.1
loguru~=0.5.3
openpyxl~=3.0.7
pandas~=1.4.4
paramiko~=2.7.2
passlib~=1.7.4
psutil~=5.9.0
pycryptodome~=3.16.0
pydantic~=1.10.2
pymongo~=3.11.4
pytest~=7.2.0
python-dotenv~=0.21.0
python_jose~=3.3.0
pytz~=2022.2.1
regex~=2022.9.13
slowapi~=0.1.7
SQLAlchemy~=1.3.17
starlette~=0.14.2
tenacity~=8.1.0
tomlkit~=0.11.6
uvicorn~=0.14.0
yapf~=0.26.0
opencv-python~=4.6.0.66
numpy~=1.21.0
Pillow~=9.2.0
websockets~=10.4
pymysql
redis
pydantic[email]
python-multipart
aiofiles
Jinja2~=3.0.1
PyYAML
natsort
```

### Dockerfile

现在我们构建一个dockerfile文件，这个文件是用来构建docker镜像。

我们可以在这个文件中指定我们的镜像基于哪个镜像，以及我们需要安装哪些依赖，以及我们需要运行哪些命令，这些都可以在dockerfile中指定。

下面是一个简单的dockerfile文件，这个文件是用来构建一个基于python3.9的镜像，这个镜像中安装了一些依赖，以及我们需要运行的命令。

```dockerfile
FROM python:3.9

ENV PYTHONUNBUFFERED 1

RUN sed -i 's/deb.debian.org/mirrors.ustc.edu.cn/g' /etc/apt/sources.list

RUN apt-get update
RUN apt-get install -y libgl1
RUN mkdir /code
WORKDIR /code
RUN pip install pip -U -i https://pypi.tuna.tsinghua.edu.cn/simple
ADD requirements.txt /code/
RUN pip install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple
ADD . /code/
ENV TZ=Asia/Shanghai
```

_1. FROM python:3.9: 指定了基础镜像，这里使用的是 Python 3.9 镜像。这意味着构建的镜像将包含 Python 3.9 环境。_

_2. ENV PYTHONUNBUFFERED 1: 设置了一个环境变量 PYTHONUNBUFFERED，将其值设置为 1。这个环境变量告诉 Python 在运行时不要对输出进行缓冲，可以实时输出。_

_3. RUN sed -i 's/deb.debian.org/mirrors.ustc.edu.cn/g' /etc/apt/sources.list: 使用 sed 命令替换 /etc/apt/sources.list 文件中的默认软件源地址 deb.debian.org 为 mirrors.ustc.edu.cn。这将加快后续的包管理操作，使用中国科技大学的镜像源。_

_4. RUN apt-get update: 使用 apt-get 命令更新软件包列表，以获取最新的可用软件包版本。_

_5. RUN apt-get install -y libgl1: 使用 apt-get 命令安装 libgl1 软件包，-y 选项表示自动回答 "yes"，无需手动确认安装。_

_6. RUN mkdir /code: 创建一个名为 /code 的目录，用于存放项目代码。_

_7. WORKDIR /code: 将当前工作目录切换为 /code 目录，即后续的指令都在该目录下执行。_

_8. RUN pip install pip -U -i https://pypi.tuna.tsinghua.edu.cn/simple: 使用 pip 命令安装最新版本的 pip 工具，并使用清华大学的镜像源进行安装。_

_9. ADD requirements.txt /code/: 将本地的 requirements.txt 文件复制到容器内的 /code 目录下。_

_10. RUN pip install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple: 使用 pip 命令根据 requirements.txt 文件安装所需的 Python 包，并使用清华大学的镜像源进行安装。_

_11. ADD . /code/: 将当前目录下的所有文件复制到容器内的 /code 目录下。_

_12. ENV TZ=Asia/Shanghai: 设置容器的时区为亚洲/上海。_

> 这个 Dockerfile 文件描述了一个基于 Python 3.9 的镜像构建过程，包括环境变量设置、软件源修改、依赖安装和代码复制等步骤。
> 可以使用该 Dockerfile 来构建一个包含所需 Python 环境和代码的 Docker 镜像。


### docker-compose.yml

docker-compose.yml 文件是用来配置 docker-compose 的，这个文件中可以指定我们需要构建哪些镜像，以及我们需要运行哪些容器，以及我们需要运行哪些命令。

下面是一个简单的 docker-compose.yml 文件，这个文件是用来构建一个基于python3.9的镜像，这个镜像中安装了一些依赖，以及我们需要运行的命令。

```yml
version: "3"
services:
  app:
    restart: always
    # docker build -t train_server_prod:2023060801 .
    image: train_server_prod:2023060801
    command: "uvicorn main:app --workers 4 --limit-concurrency 10000 --host=0.0.0.0 --port 8080"
    volumes:
      - /data:/data
      - /data2:/data2
      - /share:/share
    ports:
      - "8080:8080"
    networks:
      - web_network
  nginx:
    restart: always
    image: nginx:latest
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx:/etc/nginx/conf.d
      - ./dist:/etc/nginx/dist
      - /data/:/data/
      - /home/wutong/.acme.sh/key.key:/root/.acme.sh/key.key
      - /home/wutong/.acme.sh/cert.crt:/root/.acme.sh/cert.crt
    depends_on:
      - app
    networks:
      - web_network

networks:
  web_network:
    driver: bridge
```

这是一个使用 Docker Compose 配置的服务定义文件。它描述了两个服务：app 和 nginx。让我们逐一解释每个部分的含义：

```yml
version: "3"
services:
  app:
    restart: always
    image: train_server_prod:2023060801
    command: "uvicorn main:app --workers 4 --limit-concurrency 10000 --host=0.0.0.0 --port 8080"
    volumes:
      - /data:/data
      - /data2:/data2
      - /share:/share
    ports:
      - "8080:8080"
    networks:
      - web_network
```

- app 是一个服务名称，其中 main:app 是指运行 main.py 文件中的 app 函数。
- restart: always 指定容器在退出时总是重新启动。
- image: train_server_prod:2023060801 使用指定的镜像作为容器的基础镜像。
- command: "uvicorn main:app --workers 4 --limit-concurrency 10000 --host=0.0.0.0 --port 8080" 指定容器启动时要执行的命令。这里使用 uvicorn 来运行名为 main 的 Python 模块中的 app 应用，并配置了一些选项参数。
- volumes 部分指定了容器与主机之间的目录挂载关系。
- ports 部分将容器内部的端口映射到主机上的端口。
- networks 部分将容器连接到名为 web_network 的网络。

```yml
  nginx:
    restart: always
    image: nginx:latest
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx:/etc/nginx/conf.d
      - ./dist:/etc/nginx/dist
      - /data/:/data/
      - /home/wutong/.acme.sh/key.key:/root/.acme.sh/key.key
      - /home/wutong/.acme.sh/cert.crt:/root/.acme.sh/cert.crt
    depends_on:
      - app
    networks:
      - web_network
```

- nginx 是第二个服务的名称。
- restart: always 指定容器在退出时总是重新启动。
- image: nginx:latest 使用指定的镜像作为容器的基础镜像。
- ports 部分将容器内部的端口映射到主机上的端口。
- volumes 部分指定了容器与主机之间的目录挂载关系。
- depends_on 部分指定了 nginx 服务依赖于 app 服务。
- networks 部分将容器连接到名为 web_network 的网络。

这样，我们就可以使用 docker-compose up 命令来启动这两个服务了。
```bash
docker-compose up
```

--build 命令可以重新构建镜像。
```bash
docker compose up --build
```





