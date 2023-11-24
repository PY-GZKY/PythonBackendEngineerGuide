# 11111


## supervisor


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



## Apscheduler

`Apscheduler` 可能是作为 `fastapi` 任务队列比较理想的选择，毕竟 `Celery` 对于`fastapi` 框架而言难免过重。

> `apscheduler`: [https://apscheduler.readthedocs.io/en/stable/userguide.html](https://apscheduler.readthedocs.io/en/stable/userguide.html)

```python
$ pip install apscheduler
```

## `APScheduler` 组件：

- `触发对象`(声明要触发的类型以及参数)
- `JOB存储`(`Redis、MongoDB`)
- `执行器`(`AsyncIOExecutor`)
- `调度器`(`BackgroundScheduler、AsyncIOScheduler` )

## `Apscheduler` 模块

- `BlockingScheduler`：当调度程序是您的流程中唯一运行的东西时使用
- `BackgroundScheduler`：在不使用以下任何框架且希望调度程序在应用程序内部的后台运行时使用
- `AsyncIOScheduler`：如果您的应用程序使用`asyncio`模块，则使用
- `GeventScheduler`：如果您的应用程序使用`gevent`，则使用
- `TornadoScheduler`：在构建`Tornado`应用程序时使用
- `TwistedScheduler`：在构建`Twisted`应用程序时使用
- `QtScheduler`：在构建`Qt`应用程序时使用

`BlockingScheduler` 和 `BackgroundScheduler` 自不必说，用于非异步操作下的任务队列调度。

而 `fastapi` 是一个基于 `asyncio` 的异步框架，自然就是 `AsyncIOScheduler` 调度器啦

`TornadoScheduler` 调度器常用于 `Tornado` 框架，这也是 `Python` 另一个牛逼的异步框架，并且比 `fastapi` 更早。实际上 `fastapi` 是 `stsrlette`
的二次产物，这个我们在开篇的时候早已说过。

`TwistedScheduler` 调度器可用于 `Scrapy` 框架等基于`Twisted` 的应用程序。

## `Apscheduler` 触发器类型

- `date`：在您希望在特定时间点仅运行`一次`作业时使用
- `interval`：在您要以固定的`时间间隔`运行作业时使用
- `cron`：当您想在一天中的特定时间`定期`运行作业时使用

## `Apscheduler` 持久化

为了项目或者程序异常重启和查看最近的任务队列信息，我们应该使用持久化存储 `Apscheduler` 的任务信息，这里以 Redis 为例。

```python
# -*- coding: utf-8 -*-

import asyncio
import datetime

from apscheduler.events import EVENT_JOB_EXECUTED
from apscheduler.executors.asyncio import AsyncIOExecutor
from apscheduler.jobstores.redis import RedisJobStore
from apscheduler.schedulers.asyncio import AsyncIOScheduler
from apscheduler.triggers.interval import IntervalTrigger

# redis 存储job信息
default_redis_jobstore = RedisJobStore(
    db=2,
    jobs_key="apschedulers.default_jobs",
    run_times_key="apschedulers.default_run_times",
    host="127.0.0.1",
    port=6379,
    password=None
)

# asyncio是的调度执行器
async_executor = AsyncIOExecutor()

# 初始化scheduler，直接指定 jobstore 和 executor
init_scheduler_options = {
    "jobstores": {
        "redis": default_redis_jobstore
    },
    "executors": {
        "redis": async_executor
    },

    "job_defaults": {
        'coalesce': False,  # 是否合并执行
        'max_instances': 8  # 最大实例数
    }
}
# 创建 scheduler 实例，字典形式参入
scheduler = AsyncIOScheduler(**init_scheduler_options)

# 启动调度
scheduler.start()
```

- 我们使用了 `AsyncIOExecutor` 执行器
- 我们使用了 `AsyncIOScheduler` 调度器
- 我们使用 `redis` 命名了 `jobstores` 和 `executors`

## 准备两个被调函数

```python
def interval_func(message):
    print(f'普通函数 ==> {message} ==> {format(datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S"))}')


async def async_func(message):
    print(f'异步函数 ==> {message} ==> {format(datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S"))}')
```

这里我们准备了一个 `普通函数`和一个`异步函数`，不出意外的话对于 `AsyncIOScheduler` 而言可以正常调度这两个方法。

## 监听事件

```python
from apscheduler.events import EVENT_JOB_EXECUTED


def job_execute(event):
    """
    监听事件处理
    :param event:
    :return:
    """
    print(f"job执行: job.id => {event.job_id} \t jobstore =>{event.jobstore}")


# 这就相当于一个钩子方法(程序回调)，这里简单的打印一下信息
scheduler.add_listener(job_execute, EVENT_JOB_EXECUTED)
```

## 开启定时任务

```python
scheduler.add_job(interval_func, "interval",
                  args=["15s执行一次"],  # 方法参数
                  seconds=15,  # 时间间隔
                  id="interval_func_test",  # 指定任务 ID，如果不指定默认 UUID 指
                  jobstore="redis",  # 指定工作存储器，前面定义的 Redis，找不到工作存储将报错
                  executor="redis",  # 指定执行器，前面定义的 Redis，找不到执行器将报错
                  start_date=datetime.datetime.now(),  # now 开始
                  end_date=datetime.datetime.now() + datetime.timedelta(seconds=240)  # 结束事件，240 s后结束
                  )

print(f'任务启动 ==> 当前时间: {datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")}')

# asyncio 启动
asyncio.get_event_loop().run_forever()
```

## 总体代码

```python
# -*- coding: utf-8 -*-

import asyncio
import datetime

from apscheduler.events import EVENT_JOB_EXECUTED
from apscheduler.executors.asyncio import AsyncIOExecutor
from apscheduler.jobstores.redis import RedisJobStore  # 需要安装redis
from apscheduler.schedulers.asyncio import AsyncIOScheduler
from apscheduler.triggers.interval import IntervalTrigger

# 定义 jobstore  使用 redis 存储job信息
default_redis_jobstore = RedisJobStore(
    db=2,
    jobs_key="apschedulers.default_jobs",
    run_times_key="apschedulers.default_run_times",
    host="127.0.0.1",
    port=6379,
    password=None
)

# 定义executor 使用asyncio是的调度执行规则
async_executor = AsyncIOExecutor()

# 初始化scheduler时，可以直接指定jobstore和executor
init_scheduler_options = {
    "jobstores": {
        "redis": default_redis_jobstore
    },
    "executors": {
        "redis": async_executor
    },

    "job_defaults": {
        'coalesce': False,  # 是否合并执行
        'max_instances': 8  # 最大实例数
    }
}
# 创建scheduler
scheduler = AsyncIOScheduler(**init_scheduler_options)

# 启动调度
scheduler.start()


def job_execute(event):
    """
    :param event:
    :return:
    """
    print(f"job执行: job.id => {event.job_id} \t jobstore =>{event.jobstore}")


# 这就相当于一个钩子方法(程序回调)，这里简单的打印一下信息
scheduler.add_listener(job_execute, EVENT_JOB_EXECUTED)


def interval_func(message):
    print(f'普通函数 ==> {message} ==> {format(datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S"))}')


async def async_func(message):
    print(f'异步函数 ==> {message} ==> {format(datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S"))}')


scheduler.add_job(interval_func, "interval",
                  args=["10s执行一次"],
                  seconds=10,
                  id="interval_func_test",
                  jobstore="redis",
                  executor="redis",
                  start_date=datetime.datetime.now(),
                  end_date=datetime.datetime.now() + datetime.timedelta(seconds=240))

print(f'任务启动 ==> 当前时间: {datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")}')

asyncio.get_event_loop().run_forever()
```

控制台结果:

```text
任务启动 ==> 当前时间: 2021-05-18 16:19:02
普通函数 ==> 10s执行一次 ==> 2021-05-18 16:19:12
job执行: job.id => interval_func_test 	 jobstore =>redis
普通函数 ==> 10s执行一次 ==> 2021-05-18 16:19:22
job执行: job.id => interval_func_test 	 jobstore =>redis
普通函数 ==> 10s执行一次 ==> 2021-05-18 16:19:32
job执行: job.id => interval_func_test 	 jobstore =>redis
...
```

说明任务被正常调度，如期所料。

任务信息被存入 `Redis` 数据库，`apschedulers.default_jobs` 和 `apschedulers.default_run_times` 对应着初始化 `RedisJobStore`
时传入的 `jobs_key` 和 `run_times_key`，在 `Redis` 中以 `hash键` 存在

```python
default_redis_jobstore = RedisJobStore(
    db=2,
    jobs_key="apschedulers.default_jobs",
    run_times_key="apschedulers.default_run_times",
    host="127.0.0.1",
    port=6379,
    password=None
)
```

![任务成功redis](./images/任务成功redis.png)

一旦已定义的 `ID` 存在于 `Redis` 中，`add_job` 方法将会失效并抛出一个`异常`

```shell
Traceback (most recent call last):
  File "D:\conda3\lib\site-packages\apscheduler\schedulers\base.py", line 448, in add_job
    self._real_add_job(job, jobstore, replace_existing)
  File "D:\conda3\lib\site-packages\apscheduler\schedulers\base.py", line 872, in _real_add_job
    store.add_job(job)
  File "D:\conda3\lib\site-packages\apscheduler\jobstores\redis.py", line 77, in add_job
    raise ConflictingIdError(job.id)
apscheduler.jobstores.base.ConflictingIdError: 'Job identifier (interval_func_test) conflicts with an existing job'
```

报错信息时 `job` 已存在，也就是说任务 `ID` 冲突了。这时候要么更换 `ID`值，要么删除 `Redis` 中的原有`ID`后重新设置。

`apscheduler` 调度器同时提供了 `get_job` 和 `remove_job` 方法，参数就是任务 `ID` 和 `jobstore`

```python
if scheduler.get_job(job_id="interval_func_test", jobstore="redis"):
    print("key interval_func_test 存在 干掉它")
    scheduler.remove_job(job_id="interval_func_test", jobstore="redis")
```

这样每次运行前判断任务是否存在，存在则干掉它，就不会产生冲突了。 真是个小机灵鬼。

需要注意的是，如果 `redis`队列中有任务时，当我们重新启动 `apscheduler` 时，
`apscheduler` 会有一个任务消费的机制，详细请下面几个参数。注意另一个`hash键`记录着任务添加的`时间戳`。

- `max_instances` 最多可以`并发`多少个`实例`，可以在初始化调度器的时候设置一个`全局默认值`，添加任务时可以再单独指定
  
- `coalesce` 值为 `True` 或 `False`，如果 `True`，那么下次这个 `job` 被提交给执行器时，只会执行 `1` 次，也就是最后这次，如果为 `False`，那么会执行堆积`所有次数`（比如某时刻突然断电，再次重启时发现队列中已经堆积了`10`个任务，那么就会执行 `10` 次该任务），但是不是绝对的，下面这个参数可能影响它。

- `misfire_grace_time`：单位为秒，`misfire_grace_time` 参数则是在线程池有可用线程时会比对该 `job` 的应调度时间跟当前时间的`差值`，如果差值`小于` `misfire_grace_time` 时，调度器会再次调度该 `job`；反之该 `job` 的执行状态为 `EVENTJOBMISSED` 了，即`错过运行`。也就是: 给`executor` 一个`容错`的`超时时间`，这个时间范围内要是该跑的`还没跑完`，就别再跑了。

> `coalesce` 与 `misfire_grace_time` 可以在初始化调度器的时候设置一个全局默认值，添加任务时可以再单独指定（即覆盖）


```python

import asyncio
import datetime

from apscheduler.events import EVENT_JOB_EXECUTED
from apscheduler.executors.asyncio import AsyncIOExecutor
from apscheduler.jobstores.redis import RedisJobStore  # 需要安装redis
from apscheduler.schedulers.asyncio import AsyncIOScheduler
from apscheduler.triggers.interval import IntervalTrigger

# 定义 jobstore  使用 redis 存储job信息
default_redis_jobstore = RedisJobStore(
    db=2,
    jobs_key="apschedulers.default_jobs",
    run_times_key="apschedulers.default_run_times",
    host="127.0.0.1",
    port=6379,
    password=None
)

# 定义executor 使用asyncio是的调度执行规则
async_executor = AsyncIOExecutor()

# 初始化scheduler时，可以直接指定jobstore和executor
init_scheduler_options = {
    "jobstores": {
        "redis": default_redis_jobstore
    },
    "executors": {
        "redis": async_executor
    },

    "job_defaults": {
        'coalesce': False,  # 是否合并执行
        'max_instances': 1000, # 支持1000个实例并发
        'misfire_grace_time': 30 # 30秒的任务超时容错
    }
}
# 创建scheduler
scheduler = AsyncIOScheduler(**init_scheduler_options)

# 启动调度
scheduler.start()

print(f'任务启动 ==> 当前时间: {datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")}')
asyncio.get_event_loop().run_forever()
```

该程序启动后会根据 **`misfire_grace_time`** 和 **调度时间跟当前时间的`差值`** 做比较，如果差值大于`misfire_grace_time` ，则不执行`堆积`的任务。

同时会从队列中清除该任务。

## 关闭调度器

`scheduler.shutdown()`

默认情况下，会关闭 `job` 存储和执行器，并等待所有正在执行的 `job` 完成。如果不想等待则可以使用以下方法关闭，也就是`强制关闭`：

`scheduler.shutdown(wait=False)`

## `暂停`/`恢复`调度器

暂停调度器：
`scheduler.pause()`

恢复调度器：
`scheduler.resume()`

启动调度器的时候可以指定 `paused=True`，以这种方式启动的调度器直接就是`暂停`状态。

`scheduler.start(paused=True)`

## 共享队列
`scheduler` 默认不支持共享消息队列(`Redis`)


> 以上涉及的代码用例: [https://gitee.com/PY-GZKY/django-fastapi-docs/tree/master/code](https://gitee.com/PY-GZKY/django-fastapi-docs/tree/master/code)

> 参考文章: [https://www.cnblogs.com/leffss/p/11912364.html](https://www.cnblogs.com/leffss/p/11912364.html)
[http://sinhub.cn/2018/11/apscheduler-user-guide/](http://sinhub.cn/2018/11/apscheduler-user-guide/)