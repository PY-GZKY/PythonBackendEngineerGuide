## Centos
```shell
yum install Supervisor
```

 进入` cd /etc/supervisor` 目录 找到 `supervisord.conf` 配置文件 和 
`supervisord.d` 文件夹，编辑s`upervisord.conf`文件（默认在 `/etc/supervisor/supervisord.conf`）

 `files = supervisord.d/*.ini` 这句代码说明它会加载`supervisord.d`文件夹中的所有`.ini`配置文件

#### Start
```shell
supervisord -c /etc/supervisor/supervisord.conf
```

#### Start 某一个应用
```shell
# 重启supervisor
supervisorctl reload
# 启动名字为django的program 
supervisorctl start django
# 停止名字为django的program 
supervisorctl stop django
# 查看名字为django的program 状态
supervisorctl status django
# 查看所有program的状态
supervisorctl status all
# 停止所有program
supervisorctl stop all
```

如 xxx.ini 放于 supervisord.d 下
```shell
[program:DeployLinux]   #DeployLinux  为程序的名称
command=dotnet DeployLinux.dll #需要执行的命令
directory=/home/publish #命令执行的目录
environment=ASPNETCORE__ENVIRONMENT=Production #环境变量
user=root #用户
stopsignal=INT 
autostart=true #是否自启动
autorestart=true #是否自动重启
startsecs=3 #自动重启时间间隔（s）
stderr_logfile=/var/log/ossoffical.err.log #错误日志文件
stdout_logfile=/var/log/ossoffical.out.log #输出日志文件
```

```shell
supervisorctl reload  //重新加载配置文件
```


`supervisord.conf` 配置项中
```shell
[inet_http_server]         ; inet (TCP) server disabled by default
port=0.0.0.0:9001          ; ip_address:port specifier, *:port for all iface
username=user              ; default is no username (open server)
password=123               ; default is no password (open server)

```

```shell
firewall-cmd --query-port=9001/tcp     
firewall-cmd --add-port=9001/tcp           
firewall-cmd --reload                 
```

#### 问题不大
```shell
curl --request GET -sL \
     --url 'http://127.0.0.1:9001'
```

> PS: supervisord 依赖于 Python2, 但是 supervisor 客户端库可以用 pip3 安装



## 报错答疑
#### A：Starting supervisor: Unlinking stale socket /var/run/supervisor.sock
```shell
find / -name supervisor.sock
unlink /***/supervisor.sock
```

#### B：CentOS7：【error】Failed to start firewalld.service: Unit firewalld.service is masked

```shell
systemctl unmask firewalld.service
```
#### C: firewall-cmd
```shell
# firewall-cmd --zone=public --add-port=80/tcp --permanent    

命令含义：

--zone #作用域
--add-port=80/tcp  #添加端口，格式为：端口/通讯协议
--permanent   #永久生效，没有此参数重启后失效

从public移除 interface  
# firewall-cmd --zone=public  --remove-interface=eno16777736  

查询外网端口  
# firewall-cmd --permanent --query-port=8080/tcp  
  
删除8080端口，禁止外网访问  
# firewall-cmd --permanent --remove-port=8080/tcp   
  
添加8080端口，供外网访问  
# firewall-cmd --permanent --add-port=8080/tcp   

暂时开放ftp服务
firewall-cmd --add-service=ftp

永久开放ftp服务
firewall-cmd --add-service=ftp --permanent

永久关闭ftp服务
firewall-cmd --remove-service=ftp --permanent
```

```shell
systemctl start firewalld.service
systemctl stop firewalld.service
systemctl enable firewalld.service
systemctl disable firewalld.service
systemctl status firewalld
```

#### 基本配置项说明

```shell
[unix_http_server]：这部分设置HTTP服务器监听的UNIX domain socket
file: 指向UNIX domain socket，即file=/var/run/supervisor.sock
chmod：启动时改变supervisor.sock的权限

[inet_http_server]启动 supervisorctl 客户端后，可以用浏览器打开输入用户名、密码后，进入web界面
port=127.0.0.1:9001 ; ip_address:port specifier, *:port for all iface
username=root ; default is no username (open server)
password=123 ; default is no password (open server)

[supervisord]：与supervisord有关的全局配置需要在这部分设置
logfile=/usr/local/var/log/supervisord.log ;日志文件，默认是 $CWD/supervisord.log
logfile_maxbytes=50MB ;日志文件大小，超出会rotate，默认 50MB，如果设成0，表示不限制大小
logfile_backups=10 ;日志文件保留备份数量默认10，设为0表示不备份
loglevel=info ;日志级别，默认info，其它: debug,warn,trace
pidfile=/usr/local/supervisor-3.3.4/supervisord.pid ;pid 文件
nodaemon=false ;是否在前台启动，默认是false，即以 daemon 的方式启动
minfds=1024 ;可以打开的文件描述符的最小值，默认 1024
minprocs=200 ;可以打开的进程数的最小值，默认 200

[supervisorctl]：
serverurl：进入supervisord的URL， 对于UNIX domain sockets, 应设为 unix:///absolute/path/to/file.sock

[include]：如果配置文件包含该部分，则该部分必须包含一个files键：
files：包含一个或多个文件，这里包含了/etc/supervisor/conf.d/目录下所有的.conf文件，可以在该目录下增加我们自己的配置文件，在该配置文件中增加[program:x]部分，用来运行我们自己的程序，如下：

子程序的相关配置
[program:capital-balance] ：配置文件必须包括至少一个program，x是program名称，必须写上，不能为空
directory = /Library/WebServer/Documents/project ;执行子进程时supervisord暂时切换到该目录
command = /bin/bash -c ‘./yii 执行的命令’ ;包含一个命令，当这个program启动时执行
autostart = true
startsecs = 5
autorestart = true
startretries = 3;进程从STARING状态转换到RUNNING状态program所需要保持运行的时间（单位：秒）
user = user
process_name=%(program_name)s_%(process_num)002d ;当numprocs大于1是需要使用这个
numprocs=2
redirect_stderr = true;如果是true，则进程的stderr输出被发送回其stdout文件描述符上的supervisord
stdout_logfile_backups = 20;要保存的stdout_logfile备份的数量
stdout_logfile=/Library/WebServer/Documents/capital/backend/runtime/logs/supervisor/testPopQueue.log;将进程stdout输出到指定文件
stdout_logfile_maxbytes=10MB;stdout_logfile指定日志文件最大字节数，默认为50MB，可以加KB、MB或GB等单位
stderr_logfile=/Library/WebServer/Documents/capital/backend/runtime/logs/supervisor/testPopQueue-err.log
stderr_logfile_maxbytes=10MB
```