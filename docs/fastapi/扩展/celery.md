## celery

```shell
pip install celery（5.1.2） 
```

### start worker
```shell
celery  -A celery_app.app worker -l info -c 4  -Q main-queue  -P eventlet
```

### start beat
```shell
celery  -A celery_app.app beat -l info
```

## flower
```shell
pip install flower 
```

### start flower
```shell
celery  -A celery_app.app flower --address=127.0.0.1 --port=5566
```

```shell
启动脚本 - backend/app/celery_worker_start.sh
#linux
# CELERY 4
celery worker -A app.celery_.worker.example -l info -Q main-queue -c 1
# CELERY 5
celery  -A app.celery_.worker.example  worker -l info -Q main-queue -c 1
#win
# CELERY 4
celery worker -A app.celery_.worker.example -l info -Q main-queue -c 1  -P eventlet
# CELERY 5
celery  -A app.celery_.worker.example worker -l info -Q main-queue -c 1  -P eventlet

CMD后台启动一个或多个职程
celery multi start w1 -A {worker_name} -l info
```