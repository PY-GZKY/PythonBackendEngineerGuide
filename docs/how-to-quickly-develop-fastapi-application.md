# `FastAPI`

* 在线文档：[https://fastapi.tiangolo.com/](https://fastapi.tiangolo.com/)
* 项目地址：[https://github.com/tiangolo/fastapi](https://github.com/tiangolo/fastapi)

## `FastAPI` 是不是银弹

首先，`FastAPI` 只是说它能把「功能开发速度提升约 `200%` 至 `300%`」，距离十倍还差了一些，自然不能算是银弹。

但是这两年看到了很多对 `FastAPI` 的盲目吹捧，仿佛 `FastAPI` 就是完美的解决方案。这也许都要归功于 `FastAPI` 在其 `README`
和文档里「大胆」的措辞和承诺以及不厌其烦的特性介绍。举例来说，如果它只是说「`Very high performance, it is comparable to some frameworks in Go and NodeJS`
」，那么用户也许就会认为「和 `Go/NodeJS` 有一拼」，但是说成「`Very high performance, on par with NodeJS and Go`」，那么用户就会想「`和 Go/NodeJS 不相上下`
」（以至于中文翻译是「可与 `NodeJS` 和 `Go` 比肩的极高性能」），这当然会带来一些争议。再比如，如果只是说「`double or triple the development speed`
」，那么用户大概会想「效率还不错」，但是它用了「`Increase the speed to develop features by about 200% to 300%`」，数学不好的用户就会惊呼「哇，提高 `30`
倍开发效率」。同时这种营销带来的狂热用户，也很容易被煽动去给 `Python Steering Council` 施加压力——在 `Python 3.10` 快要发布的时候突然跳出来要求撤销一个 `PEP`。

这种营销至上的项目运营方式带来的直接后果是糟糕的维护状态。`FastAPI`
似乎在慢慢变成一个翻译项目，在代码里的文档字符串都还没写的情况下，其用户文档就已经开始扩展到十几种语言。而这十几种语言的翻译和项目源码都放在同一个仓库里，这导致所有开启的近 `300` 个 `PR`
里有过半是翻译。`issue tracker` 和 `discussion` 不做区分的使用，这导致所有开启的近 `600` 个 `issue` 里有九成是提问）。我在此前创建过一个 `issue`
来反馈这个问题，但并没有得到回应。另外凭个人喜好来说，在每个 `commit` 信息里都加上 `emoji` 并不可爱。而每一个 `commit` 都要触发 `bot` 更新 `changelog`，带来的是一份丑陋的 `commit`
历史（所有 `commit` 里有三分之一是在更新 `changelog`）。这些也许是发展社区或是让项目看起来很活跃的有效方法，但很显然这带来的是建立在混乱之上的虚假繁荣。

另外，许多推介文章都会在最后贴出来一张 `benchmark` 截图来证明 `FastAPI` 有多么 `fast`，但是完全不会提及的是：贴出来的 `benchmark` 对于开发生产应用来说有多大的意义？这里的 `benchmark`
背后有没有任何的 `hacky`？异步是否等同于高性能还是要看情况？框架本身的性能在一个请求处理流程中占多大的影响？`asyncio` 的生态怎么样？

从长远看这些大都是一些临时问题，而且 `FastAPI` 作者已经开始全职开发开源项目，这些问题在未来应该都会慢慢得到改善。指出这些是希望更多的人可以客观看待 `FastAPI`
，吹捧并不能让一个东西变得更好，参与开发、介绍用法和回答社区提问是比盲目吹捧更有意义的事情。我当然期待 `FastAPI` 能够越来越好，也期待看到有更多优秀的 `Python` 框架出现，但我不喜欢过度炒作、盲目的吹捧和错误的对比。


## 如何使用 FastAPI 连接数据库

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


