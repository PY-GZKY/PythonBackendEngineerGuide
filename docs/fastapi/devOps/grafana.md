## `Grafana` 集成进自己的监控项目

## `vue`嵌套`grafana`展示大盘数据

可能有需求是将`grafana`的`dashboard`集成到自己的监控系统里面，这样就避免了进`grafana`再查看

方案有是有，可能有点不安全，建议实在要这么干的话：

尽量是公司内部玩，也就是纯内网操作；

只有公司`IP`可以访问监控系统和`grafana`的`域名/IP`；

以下是方法：
嵌`grafana`监控`dashboard`，只需要在`web`监控`iframe`中嵌进去：

```shell
<iframe src="http://192.168.0.1:3000/d/oidoT24Wk/apache-jmeter?refresh=5s&orgId=1" width="450" height="200" frfameborder="0"></iframe>
```

`src`后面放`dashboard`的页面即可

但是这样有个问题，直接打开，会跳转到登录页面，也就是想这么做的话，需要开启匿名登录

修改`grafana`配置文件: `vim /etc/grafana/grafana.ini`

```shell
[auth.anonymous]
# enable anonymous access
# 去掉注释，改为true，允许匿名访问
enabled = true

# specify organization name that should be used for unauthenticated users
# 匿名用户属于的组织
org_name = Main Org.

# specify role for unauthenticated users
# 匿名用户的角色/权限
org_role = Viewer
```

如果提示有 `in a frame because it set 'X-Frame-Options' to 'deny'.`报错

解决方案:

`/etc/grafana/grafana.ini`配置文件修改`allow_embedding = true`

> 本文出自 https://www.dazhuanlan.com/2019/09/22/5d875a32615bc/