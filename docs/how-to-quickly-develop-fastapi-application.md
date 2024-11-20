# 如何快速开发 `FastAPI` 应用

* 在线文档：[https://fastapi.tiangolo.com/](https://fastapi.tiangolo.com/)
* 项目地址：[https://github.com/tiangolo/fastapi](https://github.com/tiangolo/fastapi)

## `FastAPI` 是不是银弹?

首先，`FastAPI` 只是说它能把「功能开发速度提升约 `200%` 至 `300%`」，距离十倍还差了一些，自然不能算是银弹。

## 如何使用 FastAPI 连接数据库?

在 Python Web 后端开发中，如果是关系型数据库交互的话，可以无脑的选择 SQLAlchemy 作为 ORM（对象关系映射）工具。

当然，如果你对 SQL 语句很熟悉，使用 PyMySQL这一类的库也是可以的。db.py 文件代码如下：

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

## 执行 ORM 对象还是原生 SQL 语句?

这个问题没有标准答案，取决于你的业务需求和个人习惯。

sqlalchemy 推荐使用 ORM 对象，可以直接操作数据表，也可以使用 execute 方法执行原生 SQL 语句。

## sqlalchemy 进行 ORM 操作时的 SQL 语句是什么?

```python
engine = create_engine(settings.SQLALCHEMY_DATABASE_URL, eoch=True, poolclass=QueuePool, pool_pre_ping=True, pool_size=20, pool_recycle=3600)
```

使用 `echo=True` 参数，可以在控制台输出 ORM 操作时的 SQL 语句，方便调试。


## 如何按需查询字段?

```python
# -*- coding: utf-8 -*
import time
from typing import Any, Optional

from fastapi import APIRouter, Depends
from sqlalchemy.orm import Session, load_only

from app import schemas
from app.api.utils.response_code import resp_200, resp_208, resp_400, resp_403
from app.common import deps
from app.db.mysql_db import get_mysql_db
from app.models import Ploy, Campaign

router = APIRouter()


@router.get("/list", response_model=schemas.Response, summary="策略管理")
def get_ploy(
        *,
        db: Session = Depends(get_mysql_db),
        limit: int,
        page: int,
        title: Optional[str] = None,
        nickname: Optional[str] = None,
        shopname: Optional[str] = None,
        token_check=Depends(deps.get_current_active_user)
) -> Any:
    """
    策略管理 查询
    """
    query = db.query(Ploy).options(
        load_only(
            Ploy.PloyId, Ploy.Title, Ploy.TjDays, Ploy.Sort, Ploy.Status,
            Ploy.Platform, Ploy.Remark, Ploy.ShopName, Ploy.Nickname, Ploy.Nickname2, Ploy.CreateTime, Ploy.EditTime
        )
    )
    if title:
        query = query.filter(Ploy.Title.ilike(f"%{title}%"))
    if nickname:
        query = query.filter(Ploy.Nickname.ilike(f"%{nickname}%"))
    if shopname:
        query = query.filter(Ploy.ShopName == shopname)

    total = query.count()  # 获取总数. 挖一个坑，后面再填

    # 根据 CreateTime 排序
    items = query.order_by(Ploy.CreateTime.desc()).limit(limit).offset((page - 1) * limit).all()

    results = []
    for item in items:
        result_item = {
            "PloyId": item.PloyId,
            "Title": item.Title,
            "TjDays": item.TjDays,
            "Sort": item.Sort,
            "Status": item.Status,
            "Platform": item.Platform,
            "Remark": item.Remark,
            "ShopName": item.ShopName,
            "Nickname": item.Nickname,
            "Nickname2": item.Nickname2,
            "CreateTime": item.CreateTime.strftime('%Y-%m-%d %H:%M:%S'),
            "EditTime": item.EditTime.strftime('%Y-%m-%d %H:%M:%S')
        }
        results.append(result_item)

    data = {"items": results, 'total': total}
    return resp_200(data=data)
```

通过操作 load_only 属性来加载所需的字段而不是全部字段，这在一定程度上可以提高查询的性能以及减少数据传输。