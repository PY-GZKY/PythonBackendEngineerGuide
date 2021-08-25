[comment]: <> (![logo]&#40;https://www.easyicon.net/api/resizeApi.php?id=1214577&size=96&#41;)

:four_leaf_clover: :four_leaf_clover:  :four_leaf_clover:  :four_leaf_clover:  :four_leaf_clover:  :four_leaf_clover:  :four_leaf_clover:  :four_leaf_clover:  :four_leaf_clover:  :four_leaf_clover:  :four_leaf_clover:  :four_leaf_clover:  :four_leaf_clover:  :four_leaf_clover:  :four_leaf_clover:  :four_leaf_clover:  :four_leaf_clover:  :four_leaf_clover:  :four_leaf_clover:  :four_leaf_clover: :pineapple: :pineapple:

> 开始一些 `Python` 相关的配置

![Python](https://img.shields.io/badge/Python-3.7-blue)
![Python](https://img.shields.io/badge/Python-3.8-red)
![Windows](https://img.shields.io/badge/Windows-10-green)
![Linux](https://img.shields.io/badge/Linux-unkown-yellow)

## `Python` 虚拟环境

成功安装 `Python` 之后默认只提供一个版本，比如 `Python3.7`

> 这对于习惯切换多版本开发的小伙伴可就不太友好了 ！！

所以，为了兼容不同`Python`版本共同开发，我们需要使用 `Python` 虚拟环境

由于我们前面是使用 `conda` 搭建的 `Python` 环境，而 `conda` 也提供了 `Python`的虚拟环境，可以切换到不同的`Python`
版本进行开发，不同版本之间相互隔离，是一个独立的操作环境 ！！

这里假设读者本地默认安装了 `Python3.7` 版本

### 创建一个新的虚拟环境

```shell
# Python 3.6  
$ conda create -n venv_3.6 python=3.6   

# Python 3.8 
$ conda create -n venv_3.8 python=3.8
```

其中 `-n` 指定了虚拟环境的名称，可以随便起，后面跟着 `Python` 和指定`版本号`

查看创建的虚拟环境列表

```shell
> conda env list
# conda environments:
#
base                  *  D:\conda3
venv_3.6                 D:\conda3\envs\venv_3.6
venv_3.8                 D:\conda3\envs\venv_3.8
```

> 可以看到刚才创建的环境已经有了

### 激活并进入虚拟环境

```shell
Linux:   source venv
Windows: activate venv
```

### 安装 `pipenv`

```shell
pip install pipenv
```

进入新的虚拟环境会发现很多第三方工具和扩展包需要重新安装，比如 `PyMysql` 等等

想往常一样执行 `pip install pymysql` 安装第三方扩展即可

### 删除一个现有的虚拟环境

```shell
# 将清空所有
conda remove --name venv --all 
```

> 注意，如果是`Pycharm`编辑器的话可以将设置中的项目控制中的`Python`解释器路径更改为 `venv` 的路径，更改生效后，这就意味着你已经进入了新的`Python`版本环境

## `FastAPI` 版本

> 不做限制
>

### 一些 `FastAPI` 开发预备库

```shell
aiohttp
aiofiles==0.5.0
alembic==1.4.2
bcrypt==3.1.7
cffi==1.14.0
click==7.1.2
dnspython==1.16.0
ecdsa==0.15
email-validator==1.1.1
fastapi==0.59.0
h11==0.9.0
httptools==0.1.1
idna==2.9
loguru==0.5.1
Mako==1.1.3
MarkupSafe==1.1.1
passlib==1.7.2
pyasn1==0.4.8
pycparser==2.20
pydantic==1.5.1
PyMySQL==0.9.3
python-dateutil==2.8.1
python-editor==1.0.4
python-jose==3.1.0
python-multipart==0.0.5
rsa==4.6
six==1.15.0
SQLAlchemy==1.3.17
starlette==0.13.4
uvicorn==0.11.5
uvloop==0.14.0
websockets==8.1
```

## `Django` 版本

虽然 `Django3.0` 已经问世，当很多开发者仍然使用 `Django2.0` 进行开发。

- 如果是`Django2.0`，这里建议选择 `Django2.2.6`
- 如果是`Django3.0`，选择 `Django3.1` 即可

### 一些 `Django` 开发预备库

```shell
celery==5.0.5
chardet==3.0.4
Django==2.2.6
django-ckeditor==5.7.1
django-debug-toolbar==2.0
django-import-export==2.0.2
django-js-asset==1.2.2
django-mdeditor==0.1.18
django-pure-pagination==0.3.0
django-redis==4.11.0
djangorestframework==3.11.0
gunicorn==20.0.4
importlib-metadata==1.4.0
Markdown==3.2.1
PyMySQL==0.9.3
redis==3.3.11
requests==2.22.0
requests-html==0.10.0
urllib3==1.25.7
websockets==8.1
```

## `FastAPI` 配置项

相对于 `Django`， `fastapi`的配置非常的灵活，你可以用任何你喜欢的命名、配置语法来书写配置文件 ！！

比如你可以用 `config.py` 或 `settings.py` 文件作为主配置文件

也可以使用诸如 `.yaml`、 `.ini` 等一些你喜欢的配置风格文件，最后解析这个文件即可。

关于配置文件的声明，我建议采用大写字符作为变量的写法，这似乎是编程开发中一种不成文的规定 ！！

当然你也可以反其道而行之，用一些小写字符和单下划线组成的变量名也不会对项目产生影响。

比如笔者的 `FastApi` 配置文件大概如此:

```shell
# -*- coding: utf-8 -*-
import os
from typing import List, Union
from databases import DatabaseURL
from pydantic import AnyHttpUrl, BaseSettings, validator, EmailStr

class Settings(BaseSettings):
    DEBUG: bool = True
    API_V1: str = "/api/v2"
    SECRET_KEY: str = "(-ASp+_)-Ulhw0848hnvVG-iqKyJSD&*&^-H3C9mqEqSl8KN-YRzRE"

    # token过期时间 60 minutes * 24 hours * 8 days = 8 days
    ACCESS_TOKEN_EXPIRE_MINUTES: int = 60 * 24

    BASE_DIR: str = os.path.dirname(os.path.dirname(os.path.dirname(os.path.abspath(__file__))))

    # 项目信息
    PROJECT_NAME: str = "桐哥"
    DESCRIPTION: str = "桐哥太困"
    SERVER_NAME: str = "API_V2"

    # 跨域
    BACKEND_CORS_ORIGINS: List[str] = ['http://localhost:8003']

    #  Redis配置项
    REDIS_USERNAME: str = os.getenv("REDIS_USERNAME", "")
    REDIS_PASSWORD: str = os.getenv("REDIS_PASSWORD", "")
    REDIS_HOST: str = os.getenv("REDIS_HOST", "")
    REDIS_PORT: int = os.getenv("REDIS_PORT", 6379)
    REDIS_DATABASE: int = os.getenv("REDIS_DATABASE", 1)
    REDIS_MAX_CONNECTIONS: int = os.getenv("REDIS_MAX_CONNECTIONS", 10)

    #  MongoDB配置项
    MAX_CONNECTIONS_COUNT = int(os.getenv("MAX_CONNECTIONS_COUNT", 10))
    MIN_CONNECTIONS_COUNT = int(os.getenv("MIN_CONNECTIONS_COUNT", 5))
    MONGODB_URL = os.getenv("MONGODB_URL", "")
    if not MONGODB_URL:
        MONGO_HOST: str = os.getenv("MONGO_HOST", "localhost")
        MONGO_PORT: int = os.getenv("MONGO_PORT", 27017)
        MONGO_USER: str = os.getenv("MONGO_USER", "")
        MONGO_PASS: str = os.getenv("MONGO_PASS", "")
        MONGO_DB: str = os.getenv("MONGO_DB", "python")
        MONGO_TABLE: str = os.getenv("MONGO_TABLE", "excel_data")
        MONGODB_URL: str = f"mongodb://{MONGO_HOST}:{MONGO_PORT}"  # 如果携带账号和密码就加上

    # MYSQL配置项
    MYSQL_USERNAME: str = os.getenv("MYSQL_USERNAME", "")
    MYSQL_PASSWORD: str = os.getenv("MYSQL_PASSWORD", "")
    MYSQL_HOST: str = os.getenv("MYSQL_HOST", "")
    MYSQL_DATABASE: str = os.getenv("MYSQL_DATABASE", "")

    # MYSQL 
    SQLALCHEMY_DATABASE_URL = f"mysql+pymysql://{MYSQL_USERNAME}:{MYSQL_PASSWORD}@{MYSQL_HOST}/{MYSQL_DATABASE}?charset=utf8mb4"

    # 远程节点主机
    SERVER_HOST_IP: str = os.getenv("SERVER_HOST_IP", "")
    SERVER_HOST_PORT: int = os.getenv("SERVER_HOST_PORT", 22)
    SERVER_HOST_PWD: str = os.getenv("SERVER_HOST_PWD", "")

settings = Settings()
```

当然你可以定义多个环境的配置文件，比如:

- `localConfig.py`
- `testConfig.py`
- `prodConfig.py`

在项目那中使用 `settings.DEBUG` 这样的格式引用即可 ！！


## `Django` 配置项

Django 的主配置文件 settings 相对固定， 有一些参数是django内置必须声明的， 当然有一些变量你不声明也没有关系， django提供了这些变量的默认值。

```shell
import os
import sys
import logging
import pymysql
from django.http import request

# 在此引入 pymysql.install_as_MySQLdb() 或者在其他 example.py 文件中引用，防止数据库连接报错
pymysql.version_info = (1, 3, 13, "final", 0)
pymysql.install_as_MySQLdb()


BASE_DIR = os.path.dirname(os.path.dirname(os.path.dirname(os.path.abspath(__file__))))

# 设置 apps, extra_apps 目录
sys.path.insert(0, os.path.join(BASE_DIR, 'apps'))


# 允许任何主机访问
ALLOWED_HOSTS = ['*']

# Application definition

INSTALLED_APPS = [
    'simpleui',  # 后台模板组件
    'import_export',
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',

    'mdeditor',
    'article',
    'pure_pagination',
]

MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]

ROOT_URLCONF = 'djangoBlog.urls'

TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [os.path.join(BASE_DIR, 'templates')],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
                'django.template.context_processors.media',
                'article.views.global_setting',
            ]

        },

    },
]

# 这里如果想使用异步服务器的话，请使用 asgi.application
WSGI_APPLICATION = 'djangoBlog.wsgi.application'

AUTH_PASSWORD_VALIDATORS = [
    {
        'NAME': 'django.contrib.auth.password_validation.UserAttributeSimilarityValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.MinimumLengthValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.CommonPasswordValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.NumericPasswordValidator',
    },
]


LANGUAGE_CODE = 'zh-hans'

TIME_ZONE = 'Asia/Shanghai'

USE_I18N = True

USE_L10N = True

USE_TZ = False  


# 静态目录
STATIC_URL = '/static/'

STATIC_ROOT = os.path.join(BASE_DIR, "STATIC")

STATICFILES_DIRS = [
    os.path.join(BASE_DIR, 'static'),
]

BASE_LOG_DIR = os.path.join(BASE_DIR, "logging")
if not os.path.exists(BASE_LOG_DIR):
    os.mkdir(BASE_LOG_DIR)


MEDIA_ROOT = os.path.join(BASE_DIR, 'uploads')
MEDIA_URL = '/media/'


LOGGING = {
    'version': 1,  # 保留的参数，默认是1
    'disable_existing_loggers': False,  # 是否禁用已经存在的logger实例
    # 日志输出格式的定义
    'formatters': {
        'standard': {  # 标准的日志格式化
            'format': '%(levelname)s %(asctime)s %(module)s %(lineno)s %(message)s'
        },
        'error': {  # 错误日志输出格式
            'format': '%(levelname)s %(asctime)s %(pathname)s %(module)s %(lineno)s %(message)s'
        },
        'simple': {
            'format': '%(levelname)s %(asctime)s %(pathname)s %(lineno)s %(message)s'
        },
        'collect': {
            'format': '%(message)s'
        }
    },

    # 处理器：需要处理什么级别的日志及如何处理
    'handlers': {
        # 将日志打印到终端
        'console': {
            'level': 'DEBUG',  # 日志级别
            'class': 'logging.StreamHandler',  # 使用什么类去处理日志流
            'formatter': 'simple'  # 指定上面定义过的一种日志输出格式
        },
        # 默认日志处理器
        'default': {
            'level': 'DEBUG',
            'class': 'logging.handlers.RotatingFileHandler',  # 保存到文件，自动切
            'filename': os.path.join(BASE_LOG_DIR, "info.log"),  # 日志文件路径
            'maxBytes': 1024 * 1024 * 100,  # 日志大小 100M
            'backupCount': 5,  # 日志文件备份的数量
            'formatter': 'standard',  # 日志输出格式
            'encoding': 'utf-8',
        },
        # 日志处理级别warn
        'warn': {
            'level': 'WARN',
            'class': 'logging.handlers.RotatingFileHandler',  # 保存到文件，自动切
            'filename': os.path.join(BASE_LOG_DIR, "warn.log"),  # 日志文件路径
            'maxBytes': 1024 * 1024 * 100,  # 日志大小 100M
            'backupCount': 5,  # 日志文件备份的数量
            'formatter': 'standard',  # 日志格式
            'encoding': 'utf-8',
        },
        # 日志级别error
        'error': {
            'level': 'ERROR',
            'class': 'logging.handlers.RotatingFileHandler',  # 保存到文件，自动切
            'filename': os.path.join(BASE_LOG_DIR, "error.log"),  # 日志文件路径
            'maxBytes': 1024 * 1024 * 100,  # 日志大小 100M
            'backupCount': 5,
            'formatter': 'error',  # 日志格式
            'encoding': 'utf-8',
        },
    },

    'loggers': {
        # 默认的logger应用如下配置
        '': {
            'handlers': ['default', 'warn', 'error'],
            'level': 'DEBUG',
            'propagate': True,  # 如果有父级的logger示例，表示不要向上传递日志流
        },
        'collect': {
            'handlers': ['console', 'default', 'warn', 'error'],
            'level': 'INFO',
        }
    },
}

# 配置新后台模板信息
SIMPLEUI_LOGIN_PARTICLES = False
SIMPLEUI_ANALYSIS = False
SIMPLEUI_STATIC_OFFLINE = True
SIMPLEUI_LOADING = False
SIMPLEUI_LOGO = 'https://xxxxxxdbf3xxxxxxxx.jpeg' 

# Github 第三方登录
GITHUB_CLIENT_ID = "xxxxxxdbf3xxxxxxxx"
GITHUB_CLIENT_SECRET = "xxxxxxxxxx0ec3d72d69007exxxxxxx"
GITHUB_CALL_ADD = "callback url"  # 回调地址

```

