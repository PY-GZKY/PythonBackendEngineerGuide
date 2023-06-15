
## Prometheus
> 从 https://prometheus.io/download/ 下载相应版本，安装到服务器上
```shell
# 解压安装包
tar -xf prometheus-2.23.0.linux-amd64.tar.gz -C /opt

# 创建链接目录
cd /opt
ln -s prometheus-2.23.0.linux-amd64 prometheus

# 直接使用默认配置文件启动
/opt/prometheus/prometheus --config.file="/opt/prometheus/prometheus.yml" &

# 确认是否正常启动（默认端口9090）
[root@server ~]# netstat -lnptu | grep 9090
tcp6       0      0 :::9090                 :::*                    LISTEN      103006/prometheus 
```
### node_exporter
```shell
# 解压安装包
tar -xf node_exporter-1.0.1.linux-amd64.tar.gz -C /opt

# 创建链接目录
cd /opt
ln -s node_exporter-1.0.1.linux-amd64 node_exporter

# 使用nohup后台运行
nohup /opt/node_exporter/node_exporter &

# 确认是否正常启动（默认端口9100）
[root@mysql01 ~]# netstat -lnptu | grep 9100
tcp6       0      0 :::9100                 :::*                    LISTEN      20716/node_exporter 

Extend: nohup命令: 如果把启动node_exporter的终端给关闭,那么进程也会
随之关闭。nohup命令会帮你解决这个问题。
```
### mysqld_exporter
```shell
# 解压安装包
tar -xf mysqld_exporter-0.12.1.linux-amd64.tar.gz -C /opt

# 创建链接目录
cd /opt
ln -s mysqld_exporter-0.12.1.linux-amd64 mysqld_exporter

# 在MySQL服务器上创建监控用户
mysql> grant select,replication client, process on *.* to 'mysql_monitor'@'localhost' identified by '123';
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.03 sec)

mysql> exit
Bye

# 将上面创建的mysql用户信息写入mysqld_exporter配置文件（新创建一个）
[root@mysql01 ~]# vim /opt/mysqld_exporter/.my.cnf
[client]
user=mysql_monitor
password=123

# 启动mysqld_exporter
nohup /opt/mysqld_exporter/mysqld_exporter --config.my-cnf=/opt/mysqld_exporter/.my.cnf &

# 确认是否正常启动（默认端口9104）
[root@mysql01 ~]# netstat -lnptu | grep 9104
tcp6       0      0 :::9104                 :::*                    LISTEN      32688/mysqld_export 
```
### prometheus.yml

```shell
在主配置文件最后面添加被监控主机信息
[root@server ~]# vim /opt/prometheus/prometheus.yml 

  - job_name: '10.0.3.105'      # 给被监控主机取个名字，我这里直接填的IP
    static_configs:
    - targets: ['10.0.3.105:9100']      # 这里填写被监控主机的IP和端口

  - job_name: '10.0.3.115'
    static_configs:
    - targets: ['10.0.3.115:9100']
    
  - job_name: 'mysql-105'      # 给被监控主机取个名字
    static_configs:
    - targets: ['10.0.3.105:9104']      # 这里填写被监控主机的IP和端口

  - job_name: 'mysql-115'
    static_configs:
    - targets: ['10.0.3.115:9104']

有多少台被监控主机就照格式添加在后面好了，我这里监控了105/115两台主机
```

```shell
[root@server ~]# pkill prometheus 
[root@server ~]# /opt/prometheus/prometheus --config.file="/opt/prometheus/prometheus.yml" &
[root@server ~]# netstat -lnptu | grep 9090
```
## Grafana

```shell
我这是使用的是CentOS系统，直接下载rpm包就好
yum localinstall grafana-7.3.5-1.x86_64.rpm 

启动服务并加入开机启动
systemctl start grafana-server.service 
systemctl enable grafana-server.service 

检查服务状态（默认使用3000端口）
systemctl status grafana-server.service 

netstat -lnptu | grep 3000
tcp6       0      0 :::3000                 :::*                    LISTEN      112219/grafana-serv 
```