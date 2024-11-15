# Ubuntu服务器 装机指南

## 如何选配一台服务器?

* CPU：8核心处理器
* 主板：Kingston A2000
* 硬盘：1T、2T 或更多
* 内存：8GB、16GB 或更多

这是一个基本的配置，具体的选择还取决于你服务器的使用情况。如果你的服务器将用于运行大型数据库、虚拟化环境或其他内存密集型任务，你可能需要更多的内存。

同样，如果预算允许，可以考虑选择更高性能的 CPU 和更大容量的 SSD。

## ssh连接服务器

```shell
ssh wutong@hostname
```

## 如何进行系统换源?

如果你的服务器来自 阿里云或者腾讯云，那自然是不用换源了。这些运营商会帮你配好。

但是如果你的服务器是本地搭建的，或者是来自国外的vps，那你就需要换成一些网络稳定的国内源。

Ubuntu 的软件源配置文件是 `/etc/apt/sources.list`。将系统自带的该文件做个备份，将该文件替换为下面内容，即可使用选择的软件源镜像。

首先备份源文件：

```shell
sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak
```

然后将 /etc/apt/sources.list 文件替换为以下内容:

```text
# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
deb https://mirrors.bfsu.edu.cn/ubuntu/ jammy main restricted universe multiverse
deb https://mirrors.bfsu.edu.cn/ubuntu/ jammy-updates main restricted universe multiverse
deb https://mirrors.bfsu.edu.cn/ubuntu/ jammy-backports main restricted universe multiverse
deb http://security.ubuntu.com/ubuntu/ jammy-security main restricted universe multiverse
```

然后apt更新一下源：

```shell
sudo apt update
```

> 这是 https://mirrors.bfsu.edu.cn 中国清华镜像源，如果你不知道其他的更好的源，那就用这个就行了。

## 如何配置 Anaconda Python 环境?

Anaconda 官方下载页面：https://www.anaconda.com/downloads

你只需要下载对应系统的安装包，然后直接运行安装包即可。如果遇到网络问题，可以使用清华大学的镜像，如

```shell
wget https://mirrors.bfsu.edu.cn/anaconda/miniconda/Miniconda3-py310_23.3.1-0-Linux-x86_64.sh
bash Miniconda3-py310_23.3.1-0-Linux-x86_64.sh -b
```

检查 miniconda 是否安装成功

```shell
conda --version
```

## 配置zsh

```shell
# 配置zsh
sh -c "$(wget https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh -O -)"

# 配置完成后一定要运行下面命令修改环境变量
~/miniconda3/bin/conda init bash
~/miniconda3/bin/conda init zsh
```

## 如何设置免密ssh登录服务器?

首先进入本地电脑当前用户的.ssh文件夹中，再打开id_rsa.pub文件并复制其中所有内容

```shell
ssh-keygen
```

此然后运行下面命令获得自己的公钥:

```shell
cat ~/.ssh/id_rsa.pub
```

使用密码登录服务器，创建~/.ssh/authorized_keys文件，并设置相应权限。

```shell
touch ~/.ssh/authorized_keys
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
nano ~/.ssh/authorized_keys

# 添加你的公钥，如 ssh-rsa AAAA...
```

并设置相应权限。把刚刚复制的密钥粘贴到~/.ssh/authorized_keys文件中，如果文件已经有内容就添加到最后一行，这样每次使用ssh登录服务器就不用再输入密码了

然后重启即可：

```shell
sudo reboot
```

## 如何把装好的环境copy到另一台机器上?

1.ssh连上服务器

2.找到配置的环境目录

3.直接rsync整个文件夹到eight2: rsync -avP 本地文件夹 用户名@远程服务器:远程地址

比如：

```shell
rsync -avp /data/wutong/miniconda3 eight2.local:/data/wutong/
```

系统会提示你输出服务器密码，输入后等待拷贝完成就行了。

这样就把一台服务器的Python环境直接拷贝到另一台服务器上了，可直接使用。

## 如何查看一台服务器当前有多少个SSH连接数?

```shell
who | grep -i pts | wc -l
```

## 如何安装 Docker?

1. 首先apt安装相关依赖包：

```shell
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
```

2. 添加 Docker 的官方 GPG 密钥：

```shell
curl -fsSL https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
```

3. 使用以下指令设置稳定版仓库:

```shell
sudo add-apt-repository \
   "deb [arch=amd64] https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu/ \
   $(lsb_release -cs) \
   stable"
```

4. 更新 apt 包索引：

```shell
sudo apt-get update
```

5. 安装最新版本的 Docker:

```shell
sudo apt-get install docker-ce docker-ce-cli containerd.io
```

6. 测试 Docker 是否安装成功，输入以下指令，打印出以下信息则安装成功:

```text
$ sudo docker run hello-world

Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
1b930d010525: Pull complete                                                                                                                                  Digest: sha256:c3b4ada4687bbaa170745b3e4dd8ac3f194ca95b2d0518b417fb47e5879d9b5f
Status: Downloaded newer image for hello-world:latest


Hello from Docker!
This message shows that your installation appears to be working correctly.


To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.


To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash


Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/


For more examples and ideas, visit:
 https://docs.docker.com/get-started/

```

7. 如果要使用 Docker 作为非 root 用户，则应考虑使用类似以下方式将用户添加到 docker 组：

```shell
sudo usermod -aG docker ai
```

8. 然后重启docker即可：

```shell
sudo systemctl restart docker
```

9. 设置docker开机自动启动：

```shell
sudo systemctl enable docker
```

这样每次开机就会自启动docker服务.

## 如何定时清理指定的目录数据?

```shell
find ./ -type d -mtime +15 -exec rm -rf {} \;
```

这会清理15天前的目录数据。


## 如何将服务添加到 Linux 开机启动项?

具体的服务文件在 `/etc/systemd/system/` 目录下，比如你的服务文件是 `my-service.service`，那么你可以使用下面的命令将服务添加到开机启动项：

```shell
sudo systemctl enable my-service.service
```

例如有一个 FastAPI 项目需要实现开机自动启动（虽然使用 Docker 会更加方便），main.py 主文件入口可能是这样的：

```python
# -*- coding: utf-8 -*-
import uvicorn

from app.utils_app import create_app

app = create_app()

if __name__ == '__main__':
    uvicorn.run(app='main:app', host="0.0.0.0", port=30002)
```

再编写一个启动脚本 `start.sh`：

```shell
#!/bin/bash

# 查找监听在30002端口上的服务的进程ID
PID=$(lsof -t -i:30002)

# 如果找到了进程，杀死它
if [ ! -z "$PID" ]; then
  echo "Stopping existing service running on port 30002 with PID: $PID"
  kill -9 $PID
fi

# 等待一小会儿确保进程已经被杀死
sleep 2

# 在此处设置 ENV_TYPE 环境变量，在 centos 中的环境变量对 systemd 服务不可见
export ENV_TYPE="prod"

# 启动新的FastAPI服务，sdadmin_py311 是一个 Anaconda 环境
echo "Starting FastAPI service..."
cd /data/project/sdadmin && nohup /root/anaconda3/envs/sdadmin_py311/bin/uvicorn main:app --workers 4 --limit-concurrency 1000 --host=0.0.0.0 --port 30002 >> ./server.log 2>&1 &

echo "FastAPI service started with new PID: $!"
```

这个脚本会先查找是否有监听在 30002 端口上的服务，如果有则杀死它，然后启动新的 FastAPI 服务。

那么如何将这个服务添加到开机启动项呢？首先创建一个服务文件 `start_server.service`：

```shell
[Unit]
Description=My Server Start Script
After=network.target

[Service]
ExecStart=/data/project/sdadmin/start_server.sh
Type=forking

[Install]
WantedBy=multi-user.target
```

这个启动文件会在网络启动后执行 `/data/project/sdadmin/start_server.sh` 脚本。`Type=forking` 表示这是一个后台服务。

然后使用下面的命令将服务添加到开机启动项：
```shell
sudo cp start_server.service /etc/systemd/system/
sudo systemctl enable start_server.service
```

确认一下服务是否已经添加到开机启动项：
```shell
sudo systemctl is-enabled start_server.service
```

执行 reload systemd 并启动服务：
```shell
sudo systemctl daemon-reload
sudo systemctl start start_server.service
```

查看系统服务状态：
```shell
sudo systemctl status start_server.service
```

可以看到服务已经启动了：
```shell
● start_server.service - My Server Start Script
   Loaded: loaded (/etc/systemd/system/start_server.service; enabled; vendor preset: enabled)
   Active: active (running) since Wed 2024-11-13 17:11:04 CST; 17min ago
 Main PID: 352685 (start_server.sh)
    Tasks: 11 (limit: 396646)
   Memory: 269.4M
   CGroup: /system.slice/start_server.service
           ├─352685 /bin/bash /data/project/sdadmin/start_server.sh
           ├─352686 /root/anaconda3/envs/sdadmin_py311/bin/python /root/anaconda3/envs/sdadmin_py311/bin/uvicorn main:app --workers 4 --limit-concurrency 1000 --host=0.0.0.0 --port 30002
           ├─352687 /root/anaconda3/envs/sdadmin_py311/bin/python -c from multiprocessing.resource_tracker import main;main(4)
           ├─352688 /root/anaconda3/envs/sdadmin_py311/bin/python -c from multiprocessing.spawn import spawn_main; spawn_main(tracker_fd=5, pipe_handle=7) --multiprocessing-fork
           ├─352689 /root/anaconda3/envs/sdadmin_py311/bin/python -c from multiprocessing.spawn import spawn_main; spawn_main(tracker_fd=5, pipe_handle=9) --multiprocessing-fork
           ├─352690 /root/anaconda3/envs/sdadmin_py311/bin/python -c from multiprocessing.spawn import spawn_main; spawn_main(tracker_fd=5, pipe_handle=11) --multiprocessing-fork
           └─352691 /root/anaconda3/envs/sdadmin_py311/bin/python -c from multiprocessing.spawn import spawn_main; spawn_main(tracker_fd=5, pipe_handle=13) --multiprocessing-fork

11月 13 17:11:02 iZ8vbaggulwyv5797qfml6Z systemd[1]: Starting My Server Start Script...
11月 13 17:11:04 iZ8vbaggulwyv5797qfml6Z start_server.sh[352521]: Starting FastAPI service...
11月 13 17:11:04 iZ8vbaggulwyv5797qfml6Z start_server.sh[352521]: FastAPI service started with new PID: 352685
11月 13 17:11:04 iZ8vbaggulwyv5797qfml6Z systemd[1]: Started My Server Start Script.
```

这样就可以实现开机自动启动 FastAPI 服务了。
