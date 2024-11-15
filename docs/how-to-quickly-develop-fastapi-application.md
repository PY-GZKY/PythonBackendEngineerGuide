# 如何快速开发 `FastAPI` 应用

* 在线文档：[https://fastapi.tiangolo.com/](https://fastapi.tiangolo.com/)
* 项目地址：[https://github.com/tiangolo/fastapi](https://github.com/tiangolo/fastapi)

## `FastAPI` 是不是银弹?

首先，`FastAPI` 只是说它能把「功能开发速度提升约 `200%` 至 `300%`」，距离十倍还差了一些，自然不能算是银弹。

## 如何使用 FastAPI 连接数据库?

在 Python Web 后端开发中，如果是关系型数据库交互的话，可以无脑的选择 SQLAlchemy 作为 ORM（对象关系映射）工具。

当然，如果你对 SQL 语句很熟悉，使用 PyMySQL这一类的库也是可以的。

```python
from typing import Generator

from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker
from sqlalchemy.pool import QueuePool

from app.config import settings

engine = create_engine(settings.SQLALCHEMY_DATABASE_URL, poolclass=QueuePool, pool_pre_ping=True, pool_size=20, pool_recycle=3600)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)


def get_mysql_db() -> Generator:
    try:
        db = SessionLocal()
        yield db
    finally:
        db.close()
```

- `SQLALCHEMY_DATABASE_URL` 是数据库的连接地址，可以是 SQLite、MySQL、PostgreSQL 等。
- `poolclass=QueuePool` 是设置数据库连接池的类型，`QueuePool` 是一种线程安全的连接池。
- `pool_pre_ping=True` 是设置数据库连接池的预检测，当连接池中的连接在使用前被检测到失效时，会被自动移除。
- `pool_size=20` 是设置数据库连接池的大小，即最大连接数。
- `pool_recycle=3600` 是设置数据库连接池的回收时间，即连接在被放回连接池前的最大生存时间。
- `SessionLocal` 是一个 `sessionmaker` 对象，用于创建数据库会话。

利用 get_mysql_db 生成器函数，可用于获取数据库会话。

> 由于 create_engine 自带了连接池，所以不需要手动关闭数据库连接。


