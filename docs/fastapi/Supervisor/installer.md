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