[comment]: <> (![logo]&#40;https://www.easyicon.net/api/resizeApi.php?id=1214577&size=96&#41;)

:clap: :clap: :clap:  :clap:  :clap:  :clap:  :clap:  :clap:  :clap:  :clap:   :clap:  :clap:  :clap:  :clap:  :clap:  :clap:  :clap:  :clap:  :clap:  :point_down: :point_down:

> 在开始之前，我们需要快速安装一些服务


![Python](https://img.shields.io/badge/Python-3.7+-blue)
![Django](https://img.shields.io/badge/Django-3.1+-brightgreen)
![Nginx](https://img.shields.io/badge/Nginx-1.16.1-red)
![Docker](https://img.shields.io/badge/Docker-latest-green)
![Mysql](https://img.shields.io/badge/Mysql-5.7-blue)


## 安装`Anaconda`

不管是`Windows`或`Linux(Mac)` 系统都推荐使用`Anaconda`搭建`Python`环境，这是一个绝佳的选择。

`Windows` 下安装 `Anaconda` 看起来只需要傻瓜式操作，最后再把它放入环境变量即可，这里不在详述 :smile:

### `Linux` 下安装`Anaconda`

这里以`Centos7`为例
```shell
$ yum update &&
  yum -y install wget &&
  wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh &&
  bash ./Miniconda3-latest-Linux-x86_64.sh
```

由于某些原因，默认的`anaconda` 站点`wget`可能会比较慢，可以选择国内的镜像源平台进行安装，比如清华源就很优秀。

> [清华镜像站](https://mirrors.tuna.tsinghua.edu.cn/anaconda/miniconda/)

选择你适应的平台和系统版本，`wget` 下载至本地即可。

```shell
$ wget https://mirrors.tuna.tsinghua.edu.cn/anaconda/archive/Anaconda3-2019.03-Linux-x86_64.sh
```

至于安装步骤，一路按确认键即可，配置环境变量并让它生效

```shell
vi /etc/profile
```
在文件最尾端加入一下路径，比如我把安装目录放在了 `root` 下

```shell
PATH=$PATH:/root/anaconda3/bin
export PATH
```
```shell
$ source /etc/profile
```

查看当前的 Python 环境
```shell
[root@gzky_gz ~] conda --version
conda 4.9.2
[root@gzky_gz ~] python
Python 3.7.9 (default, Aug 31 2020, 12:42:55) 
[GCC 7.3.0] :: Anaconda, Inc. on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> 

```

> 至此，Anaconda安装完毕 ！！


## 安装`MySQL`

笔者认为`Windows` 并不适合安装 `MySQL5.7`  :smile: 推荐 `MySQL8.0`
安装过程选择界面化安装即可，你甚至不必动用 `cmd` 命令来启动 `MySQL`，只需要把 `MYSQL` 加到 `Windows` 服务并设置开机即可，这里不在详述


### `Linux` 安装 `MySQL5.7` 
> 你知道，我总是愿意用 `Centos` 来代表 `Linux` 系统，我认为它是最好的 `Linux` 服务器 ！！



[MySQL 5.7 下载](https://cdn.mysql.com//Downloads/MySQL-5.7/mysql-5.7.29-1.el7.x86_64.rpm-bundle.tar)


进入下载目录解压文件
```shell
tar -xvf mysql-5.7.29-1.el7.x86_64.rpm-bundle.tar
```

查看原有 `MySQL` 版本，避免 `MySQL` 残留，如果是第一次安装直接忽略即可。
```shell
rpm -qa|grep mariadb
```
删除`MySQL` 残留，不存在则不用管它。
```shell
rpm -e --nodeps mariadb-libs-5.5.60-1.el7_5.x86_64
```

赋予该目录可执行权限，建议 `644` 即可，别一上来就 `777`，这很危险。
```shell
chmod -R 644 mysql
```



`rpm` 安装 `MySQL` 依赖包

```shell
rpm -ivh mysql-community-common-5.7.29-1.el7.x86_64.rpm 
rpm -ivh mysql-community-libs-5.7.29-1.el7.x86_64.rpm 
rpm -ivh mysql-community-client-5.7.29-1.el7.x86_64.rpm 
rpm -ivh mysql-community-server-5.7.29-1.el7.x86_64.rpm 
```

如果安装过程中报了 `GPG keys` 的错误

请在命令后面加上 `--force --nodeps` 参数，重新安装即可。


### `MySQL` 配置文件
 
进入`/etc/my.cnf`并加上如下语句

```shell
skip-grant-tables
character_set_server=utf8
init_connect='SET NAMES utf8'
```

`注意第一行配置了免密登陆，配置成功后记得注释掉再重启 MySQL，否则将后悔莫及`

`注意第一行配置了免密登陆，配置成功后记得注释掉再重启 MySQL，否则将后悔莫及`

`注意第一行配置了免密登陆，配置成功后记得注释掉再重启 MySQL，否则将后悔莫及`



启动`MySQL`服务

```shell
$ systemctl start mysqld.service
```

此时黑窗口键入 `mysql` 可成功进入 `mysql` 客户端

设置宿主机密码，注意是本地密码而非远程连接密码
```shell
update mysql.user set authentication_string=password('123456') where user='root';
flush privileges;
```

推出 `mysql` 客户端并 `stop` 服务

```shell
$ systemctl stop mysqld.service
```

回到 `/etc/my.cnf` 配置文注释掉免密登陆的那一行，并重新启动 `mysql` 服务

```shell
$ systemctl start mysqld.service
```

发现再次进入客户端需要指定用户名密码登陆，也就是我们上面设置的用户密码了。

当然了，`123456` 这个密码连你看起来都有些不太好，`mysql` 在密码设置方面做了一些限制，一些不符合预期的密码不会被通过 ！！

如果你执意使用简单(弱密码)的话，请做下全局设置:

```shell
set global validate_password_policy=LOW;
set global validate_password_length=4;
flush privileges;
```

### 暴露 `3306` 端口

```shell
firewall-cmd --list-ports
```

```shell
firewall-cmd --zone=public --add-port=3306/tcp --permanent
firewall-cmd --reload
```

如果你使用的是一些云服务商的服务器(比如阿里云、腾讯云)，请登陆控制台设置开放 `3306` 端口即可

### 允许远程登陆
我们并不会每次都登陆云服务器去进行数据库操作，这就需要允许其他机器远程登陆宿主机。

```shell
grant all privileges on *.* to 'root'@'%' identified by '654321' with grant option;
flush privileges;
```

> 注意这里的密码有别于前面设置的宿主机本地登陆的密码。

> 至此，`MySQL` 部署完毕 ！！


## 安装 `Nginx`

### `Centos` 安装 `Nginx`

安装所需依赖
```shell
yum install gcc-c++
yum install -y pcre pcre-devel
yum install -y zlib zlib-devel
yum install -y openssl openssl-devel
```

`wget` 下载 `Nginx`, 版本号自己指定，一般为 `nginx-1.16.0 + `

```shell
$ wget -c https://nginx.org/download/nginx-1.16.1.tar.gz
```

解压、编译、安装
```shell
tar -zxvf nginx-1.16.1.tar.gz
cd nginx-1.16.1
./configure
make
make install
```
默认安装路径 `/usr/local/nginx/`

### 启动、停止 nginx

```shell
cd /usr/local/nginx/sbin/

./nginx   # 启动
./nginx -s stop  # 停止，直接查找nginx进程id再使用kill命令强制杀掉进程
./nginx -s quit  # 退出停止，等待nginx进程处理完任务再进行停止
./nginx -s reload  # 重启，新配置即可生效
```

查看`Nginx`是否正常工作

```shell
[root@gzky_gz ~] ps -ef | grep nginx
root     12375 29482  0 16:43 pts/0    00:00:00 grep --color=auto nginx
root     26388 26237  0 4月03 ?       00:00:00 nginx: master process /usr/sbin/nginx
root     26389 26218  0 4月03 ?       00:00:00 nginx: master process /usr/sbin/nginx
33       26390 26388  0 4月03 ?       00:00:00 nginx: worker process
33       26391 26388  0 4月03 ?       00:00:00 nginx: worker process
33       26392 26389  0 4月03 ?       00:00:00 nginx: worker process
33       26393 26389  0 4月03 ?       00:00:00 nginx: worker process
root     29961     1  0 3月30 ?       00:00:00 nginx: master process ./nginx
nobody   29962 29961  0 3月30 ?       00:00:00 nginx: worker process
```
> 没有问题 ！！
> 
> 默认监听 `80` 端口， 访问 `http://IP` 出现 Welcome 界面，至此，`Nginx` 安装完毕 ！！

## 安装 `Docker`

### `Centos` 一键安装`Docker ` 

````shell
$ curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
````

### 启动 `Docker`

```shell
$ systemctl start docker
```

查看 `Docker` 版本
```shell
$ docker --version
```

`Docker` 有一个有趣的镜像用于验证 `Docker` 是否正常工作
```shell
$ docker run hello-world
```

> 成功输出运行信息，至此，`Docker` 部署完毕 ！！