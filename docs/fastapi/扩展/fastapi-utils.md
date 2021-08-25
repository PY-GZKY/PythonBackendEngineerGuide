> `fastapi-utils`: [https://github.com/dmontagu/fastapi-utils](https://github.com/dmontagu/fastapi-utils)
> `fastapi-utils-docs`: [https://fastapi-utils.davidmontague.xyz](https://fastapi-utils.davidmontague.xyz)

## `Github` 前辈已经开始编写 `fastapi` 扩展

`fastapi-utils` 文档描述

- 基于`类的视图` (`cbv`)：不再在相关端点的签名中反复重复相同的依赖项。
- `响应模型`推断路由器：让`FastAPI` `response_model` 根据您的返回类型注释来推断要使用的。
- `重复任务`：在服务器启动时轻松触发`定期任务`
- 计时`中间件`：记录每个请求的基本计时信息
- `SQLAlchemy` `Sessions`：`FastAPISessionMaker` 该类提供了易于定制的`SQLAlchemy Session`依赖项
- `OpenAPI Spec`简化：简化您的`OpenAPI`操作ID，以便从`OpenAPI Generator`进行更清晰的输出


在 `Django` 中有 x`xx 扩展，实现了`类的视图`和序列化，
`fastapi-utils`包初步实现了一些扩展功能，虽然看起来并不是最佳，但确实对实际开发有很大的帮助。

### 普通的 `curd`
```python
from typing import NewType, Optional
from uuid import UUID

import sqlalchemy as sa
from fastapi import Depends, FastAPI, Header, HTTPException
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import Session
from starlette.status import HTTP_403_FORBIDDEN, HTTP_404_NOT_FOUND

from fastapi_utils.api_model import APIMessage, APIModel
from fastapi_utils.guid_type import GUID

# Begin setup
UserID = NewType("UserID", UUID)
ItemID = NewType("ItemID", UUID)

Base = declarative_base()


class ItemORM(Base):
    __tablename__ = "item"

    item_id = sa.Column(GUID, primary_key=True)
    owner = sa.Column(GUID, nullable=False)
    name = sa.Column(sa.String, nullable=False)


class ItemCreate(APIModel):
    name: str


class ItemInDB(ItemCreate):
    item_id: ItemID
    owner: UserID


def get_jwt_user(authorization: str = Header(...)) -> UserID:
    """ Pretend this function gets a UserID from a JWT in the auth header """


def get_db() -> Session:
    """ Pretend this function returns a SQLAlchemy ORM session"""


def get_owned_item(session: Session, owner: UserID, item_id: ItemID) -> ItemORM:
    item: Optional[ItemORM] = session.query(ItemORM).get(item_id)
    if item is not None and item.owner != owner:
        raise HTTPException(status_code=HTTP_403_FORBIDDEN)
    if item is None:
        raise HTTPException(status_code=HTTP_404_NOT_FOUND)
    return item


# End setup
app = FastAPI()


@app.post("/item", response_model=ItemInDB)
def create_item(
    *,
    session: Session = Depends(get_db),
    user_id: UserID = Depends(get_jwt_user),
    item: ItemCreate,
) -> ItemInDB:
    item_orm = ItemORM(name=item.name, owner=user_id)
    session.add(item_orm)
    session.commit()
    return ItemInDB.from_orm(item_orm)


@app.get("/item/{item_id}", response_model=ItemInDB)
def read_item(
    *,
    session: Session = Depends(get_db),
    user_id: UserID = Depends(get_jwt_user),
    item_id: ItemID,
) -> ItemInDB:
    item_orm = get_owned_item(session, user_id, item_id)
    return ItemInDB.from_orm(item_orm)


@app.put("/item/{item_id}", response_model=ItemInDB)
def update_item(
    *,
    session: Session = Depends(get_db),
    user_id: UserID = Depends(get_jwt_user),
    item_id: ItemID,
    item: ItemCreate,
) -> ItemInDB:
    item_orm = get_owned_item(session, user_id, item_id)
    item_orm.name = item.name
    session.add(item_orm)
    session.commit()
    return ItemInDB.from_orm(item_orm)


@app.delete("/item/{item_id}", response_model=APIMessage)
def delete_item(
    *,
    session: Session = Depends(get_db),
    user_id: UserID = Depends(get_jwt_user),
    item_id: ItemID,
) -> APIMessage:
    item = get_owned_item(session, user_id, item_id)
    session.delete(item)
    session.commit()
    return APIMessage(detail=f"Deleted item {item_id}")
```

### 基于类的视图 `cbv`
```python
from typing import NewType, Optional
from uuid import UUID

import sqlalchemy as sa
from fastapi import Depends, FastAPI, Header, HTTPException
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import Session
from starlette.status import HTTP_403_FORBIDDEN, HTTP_404_NOT_FOUND

from fastapi_utils.api_model import APIMessage, APIModel
from fastapi_utils.cbv import cbv
from fastapi_utils.guid_type import GUID
from fastapi_utils.inferring_router import InferringRouter

# Begin Setup
UserID = NewType("UserID", UUID)
ItemID = NewType("ItemID", UUID)

Base = declarative_base()


class ItemORM(Base):
    __tablename__ = "item"

    item_id = sa.Column(GUID, primary_key=True)
    owner = sa.Column(GUID, nullable=False)
    name = sa.Column(sa.String, nullable=False)


class ItemCreate(APIModel):
    name: str
    owner: UserID


class ItemInDB(ItemCreate):
    item_id: ItemID


def get_jwt_user(authorization: str = Header(...)) -> UserID:
    """ Pretend this function gets a UserID from a JWT in the auth header """


def get_db() -> Session:
    """ Pretend this function returns a SQLAlchemy ORM session"""


def get_owned_item(session: Session, owner: UserID, item_id: ItemID) -> ItemORM:
    item: Optional[ItemORM] = session.query(ItemORM).get(item_id)
    if item is not None and item.owner != owner:
        raise HTTPException(status_code=HTTP_403_FORBIDDEN)
    if item is None:
        raise HTTPException(status_code=HTTP_404_NOT_FOUND)
    return item


# End Setup
app = FastAPI()
router = InferringRouter()  # Step 1: Create a router


@cbv(router)  # Step 2: Create and decorate a class to hold the endpoints
class ItemCBV:
    # Step 3: Add dependencies as class attributes
    session: Session = Depends(get_db)
    user_id: UserID = Depends(get_jwt_user)

    @router.post("/item")
    def create_item(self, item: ItemCreate) -> ItemInDB:
        # Step 4: Use `self.<dependency_name>` to access shared dependencies
        item_orm = ItemORM(name=item.name, owner=self.user_id)
        self.session.add(item_orm)
        self.session.commit()
        return ItemInDB.from_orm(item_orm)

    @router.get("/item/{item_id}")
    def read_item(self, item_id: ItemID) -> ItemInDB:
        item_orm = get_owned_item(self.session, self.user_id, item_id)
        return ItemInDB.from_orm(item_orm)

    @router.put("/item/{item_id}")
    def update_item(self, item_id: ItemID, item: ItemCreate) -> ItemInDB:
        item_orm = get_owned_item(self.session, self.user_id, item_id)
        item_orm.name = item.name
        self.session.add(item_orm)
        self.session.commit()
        return ItemInDB.from_orm(item_orm)

    @router.delete("/item/{item_id}")
    def delete_item(self, item_id: ItemID) -> APIMessage:
        item = get_owned_item(self.session, self.user_id, item_id)
        self.session.delete(item)
        self.session.commit()
        return APIMessage(detail=f"Deleted item {item_id}")


app.include_router(router)
```


## `fastapi_utils.inferring_router`

实际意义不大

## `fastapi_utils.tasks`

用于实现 `fastapi` 定时任务

```python
from fastapi import FastAPI
from sqlalchemy.orm import Session

from fastapi_utils.session import FastAPISessionMaker
from fastapi_utils.tasks import repeat_every

database_uri = f"sqlite:///./test.db?check_same_thread=False"
sessionmaker = FastAPISessionMaker(database_uri)

app = FastAPI()


def remove_expired_tokens(db: Session) -> None:
    """ Pretend this function deletes expired tokens from the database """


@app.on_event("startup")
@repeat_every(seconds=60 * 60)  # 1 hour
def remove_expired_tokens_task() -> None:
    with sessionmaker.context_session() as db:
        remove_expired_tokens(db=db)
```

使用装饰器的方式给路由方法上了一个`定时器`。

通过传递 `seconds=60 * 60`，我们确保装饰的函数`每小时`调用一次。

- 包装的函数不应带有任何必需的参数。
- `repeat_every`函数与`async def`和`def`函数都可以正常使用。
- `repeat_every`可以安全地与 `def` 执行阻塞 `IO` 的函数一起使用-它们在`线程池`中执行（就像 `def` 端点一样）。


以下是有关的各种关键字参数的更详细说明`repeat_every`：

- `seconds`: float ：连续两次呼叫之间等待的秒数
- `wait_first`: bool = False：如果False（默认值），则在第一次调用修饰函数时立即调用包装函数。如果为True，则装饰函数将等待一个时间段，然后再对包装函数进行第一次调用
- `logger`: Optional[logging.Logger] = None ：如果您通过记录器，那么在重复执行循环中引发的任何异常都将记录（带有回溯）到提供的记录器中。
- `raise_exceptions`: bool = False
    - 如果False（默认值），则异常会在重复执行循环中捕获，但不会引发。如果您保留此参数False，则可能需要提供一个，logger以确保重复发生的事件不会只是默默地失败。
    - 如果为True，则会引发异常。为了处理这个异常，你需要注册一个异常处理程序，它能够抓住它。例如，你可以使用`asyncio.get_running_loop()`.set_exception_handler(...)，如记录 在这里。
    - 请注意，如果引发异常，则重复执行将停止。
- `max_repetitions`: Optional[int] = None：如果None（默认值），修饰的功能将永远重复。否则，它将在指定次数的调用后停止重复执行

需要知道的是，`repeat_every` 方法只能进行定时(时间段)`轮询任务`，并没有提供定时(指定准确时间)启动任务的功能，比如 我能能让程序每 `10分钟` 查询一下 `Redis` 数据库中的 `key` 是否过期，却不能在每天的 `8点` 准时执行`爬虫`任务。

所以我们仍需借助其他任务队列来满足实际需求，它们可能是：
- `Apscheduler` 
- `Celery` `分布式队列`。

这个我们后面会讲到，将 `Apscheduler` 或 `Celery` 框架融入 `fastapi` 中。


## `fastapi_utils.timing`

`fastapi_utils.timing`模块提供了基本的性能分析功能，可用于查找性能瓶颈，监视回归等。

此模块当前提供两个公共功能：

- `add_timing_middleware`，可用于向应用添加中间件，该中间件FastAPI将为每个请求记录非常基本的配置信息（开销很小）。

- `record_timing`，可以在 安装了计时中间件`starlette.requests.Request`的`FastAPI`应用程序的实例上调用（通过`add_timing_middleware`），并在调用该请求的位置发出该请求的性能信息。


```python
import asyncio
import logging

from fastapi import FastAPI
from starlette.requests import Request
from starlette.staticfiles import StaticFiles
from starlette.testclient import TestClient

from fastapi_utils.timing import add_timing_middleware, record_timing

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

app = FastAPI()
add_timing_middleware(app, record=logger.info, prefix="app", exclude="untimed")
static_files_app = StaticFiles(directory=".")
app.mount(path="/static", app=static_files_app, name="static")


@app.get("/timed")
async def get_timed() -> None:
    await asyncio.sleep(0.05)


@app.get("/untimed")
async def get_untimed() -> None:
    await asyncio.sleep(0.1)


@app.get("/timed-intermediate")
async def get_with_intermediate_timing(request: Request) -> None:
    await asyncio.sleep(0.1)
    record_timing(request, note="halfway")
    await asyncio.sleep(0.1)
```

> 这个模块看起来功能单一，似乎作用大不。放心，后面我们会继续定制我们的 `fastapi` 中间件。

## `fastapi_utils.sessions`

除了基于`类的视图`，你也可能需要重点关注一下这个`数据库会话`的扩展。

> 正在使用中 ...





