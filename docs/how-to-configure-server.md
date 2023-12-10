# Ubuntu装机指南

## 单台服务器硬件建议

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

## 换源

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

## 配置 Python 环境

Anaconda 官方下载页面：https://www.anaconda.com/downloads

你只需要下载对应系统的安装包，然后直接运行安装包即可

如果遇到网络问题，可以使用清华大学的镜像，如

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

## 设置免密ssh登录服务器

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

## 如何把装好的环境copy到另一台机器上

1.ssh连上服务器

2.找到配置的环境目录

3.直接rsync整个文件夹到eight2: rsync -avP 本地文件夹 用户名@远程服务器:远程地址

比如：

```shell
rsync -avp /data/wutong/miniconda3 eight2.local:/data/wutong/
```

系统会提示你输出服务器密码，输入后等待拷贝完成就行了。

这样就把一台服务器的Python环境直接拷贝到另一台服务器上了，可直接使用。

## 如何查看一台服务器当前有多少个SSH连接数

```shell
who | grep -i pts | wc -l
```

## 安装Docker

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

10. portainer

Portainer 是一个轻量级的管理界面，它允许您轻松地管理不同的 Docker 环境。Portainer 提供了一个直观的图形用户界面（GUI），通过它，用户可以管理 Docker 容器、镜像、网络和卷，而不需要使用 Docker
命令行工具。Portainer 适用于管理单个本地 Docker 环境，也可以用来管理多个 Docker 主机或 Swarm 集群。

使用 Portainer，您可以：

- 查看、创建、删除和管理容器。
- 查看、拉取和管理镜像。
- 创建、编辑和管理网络。
- 创建、编辑和删除卷，以及管理数据持久化。
- 查看容器的实时日志。
- 通过控制台与容器进行交互。
- 管理 Docker Swarm 或 Kubernetes 集群。
- 管理用户和团队的访问权限。
- 部署应用程序模板或使用Docker Compose。

Portainer 是为了简化 Docker 管理任务而设计的，它特别适合那些不熟悉 Docker 命令行或寻求更简单方法来管理容器的用户。

```shell
 docker run -d -p 9000:9000 --restart=always -v /var/run/docker.sock:/var/run/docker.sock --name portainer portainer/portainer
```

一旦容器启动，您可以通过浏览器访问 http://<宿主机IP>:9000 来使用 Portainer 的管理界面。如果这是第一次运行 Portainer，它会提示您创建一个管理员用户。