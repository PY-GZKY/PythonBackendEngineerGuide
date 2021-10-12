`原来 Centos 在使用 ssh 连接的时候, 会先做一个 DNS 检测, 这就拖慢的连接的速度, 我们只要把 DNS 检测关掉就可以了`

```shell
vim /etc/ssh/sshd_config
```
把 `UseDNS` 后面的 `yes` 改成 `no`

```shell
systemctl restart sshd
```

> 然后再试试`ssh连接`, 发现速度飞起!
