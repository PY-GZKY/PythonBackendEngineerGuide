
> `ASGI`（异步服务器网关接口）是`WSGI`的精神继承者，旨在在具有异步功能的`Python` `Web`服务器，框架和应用程序之间提供标准接口。

### `ASGI` 服务器
- `uvicorn`
- `Hypercorn`
- `Daphne`

### `ASGI` 框架
- `Starlette`
- `FastAPI`
- `Quart`：`Quart` 是一个类似于 `Flask` 的 `ASGI Web` 框架。
- `Tornado`
- `Sanic`
- `Django` `Channels`
- `Django3.x`

### 异步库
- `TypeSystem`：数据验证和表单渲染
- `Databases`：异步数据库。
- `ORM`：同时支持同步和异步的`ORM`。
- `HTTPX`：异步`HTTP`客户端，支持调用`ASGI`应用程序（用作测试客户端）。这是一个`http` 请求库（常见于爬虫领域）


为了了解`ASGI`的真正工作原理，让我们看一下`ASGI`规范：

`ASGI`包含两个不同的组件：

- 一个协议服务器，其终止`Socket`协议，并将它们转换成连接和每个连接的事件消息。
- 一个应用程序，驻留在协议服务器中，每个连接中实例化一次，并在事件消息发生时对其进行处理。

实际上，这就是`ASGI`应用程序的全部特性– 可调用的。同样，此可调用对象的形式由`ASGI`规范定义。格式如下：
```python
async def app(scope, receive, send):
    assert scope['type'] == 'http'
    ...
```

解释一下三个参数：

- `scope`是一个字典，其中包含有关传入请求的信息。这是一个上下文环境。其数据与`HTTP`和`WebSocket`连接间的数据有所不同。

- `receive`是用于接收`ASGI`事件消息的异步函数。

- `send` 是用于发送`ASGI`事件消息的异步函数。



> `uvicorn`: [https://github.com/encode/uvicorn](https://github.com/encode/uvicorn)    `uvicorn.org`: [https://www.uvicorn.org/](https://www.uvicorn.org/)


> Uvicorn currently supports HTTP/1.1 and WebSockets. Support for HTTP/2 is planned. 暂时不支持 Http2.0

#### `example.py`

```python
async def app(scope, receive, send):
    assert scope['type'] == 'http'

    await send({
        'type': 'http.response.start',
        'status': 200,
        'headers': [
            [b'content-type', b'text/plain'],
        ],
    })
    await send({
        'type': 'http.response.body',
        'body': b'Hello, world!',
    })
```

```shell
$ uvicorn example:app
```

实现一个最简单的 `app` 应用程序并使用 `uvicorn` 开启了了 `web` 服务，我们只是遵循了`ASGI`规范定义参数，并发送了一些数据。

在这里，我们用`send()`向客户端发送HTTP响应：首先发送`Header`，然后发送响应主体。

现在，就有了所有这些字典和原始字节数据，我将承认使用原生的ASGI并不是很易用。

幸运的是，还有更高级的选择--是时候开始谈论`Starlette`。

`Starlette`是一个优秀的项目，其中的`IMO`是`ASGI`生态系统的基础。

简而言之，它提供了一个高级组件的工具箱，例如请求和响应，可用于抽象化ASGI的某些细节。


> `starlette`: [https://github.com/encode/starlette](https://github.com/encode/starlette)

`fastapi` 基于 `starlette` 构建，`starlette` 是一个优雅的 `ASGI` 框架，上面的入口应用程序都在 `starlette` 框架中封装。

#### `example.py`
```python
from starlette.responses import PlainTextResponse

async def app(scope, receive, send):
    assert scope['type'] == 'http'
    response = PlainTextResponse('Hello, world!')
    await response(scope, receive, send)
```

```shell
$ uvicorn example:app
```

### 实例化封装调用
```python
from starlette.applications import Starlette
from starlette.responses import JSONResponse
from starlette.routing import Route


async def homepage(request):
    return JSONResponse({'hello': 'world'})

routes = [
    Route("/", endpoint=homepage)
]

app = Starlette(debug=True, routes=routes)
```

```shell
$ uvicorn example:app
```

```python
import typing

from starlette.datastructures import State, URLPath
from starlette.exceptions import ExceptionMiddleware
from starlette.middleware import Middleware
from starlette.middleware.base import BaseHTTPMiddleware
from starlette.middleware.errors import ServerErrorMiddleware
from starlette.routing import BaseRoute, Router
from starlette.types import ASGIApp, Receive, Scope, Send


class Starlette:
    """
    Creates an application instance.
    **Parameters:**
    * **debug** - Boolean indicating if debug tracebacks should be returned on errors.
    * **routes** - A list of routes to serve incoming HTTP and WebSocket requests.
    * **middleware** - A list of middleware to run for every request. A starlette
    application will always automatically include two middleware classes.
    `ServerErrorMiddleware` is added as the very outermost middleware, to handle
    any uncaught errors occurring anywhere in the entire stack.
    `ExceptionMiddleware` is added as the very innermost middleware, to deal
    with handled exception cases occurring in the routing or endpoints.
    * **exception_handlers** - A dictionary mapping either integer status codes,
    or exception class types onto callables which handle the exceptions.
    Exception handler callables should be of the form
    `handler(request, exc) -> response` and may be be either standard functions, or
    async functions.
    * **on_startup** - A list of callables to run on application startup.
    Startup handler callables do not take any arguments, and may be be either
    standard functions, or async functions.
    * **on_shutdown** - A list of callables to run on application shutdown.
    Shutdown handler callables do not take any arguments, and may be be either
    standard functions, or async functions.
    """

    def __init__(
        self,
        debug: bool = False,
        routes: typing.Sequence[BaseRoute] = None,
        middleware: typing.Sequence[Middleware] = None,
        exception_handlers: typing.Dict[
            typing.Union[int, typing.Type[Exception]], typing.Callable
        ] = None,
        on_startup: typing.Sequence[typing.Callable] = None,
        on_shutdown: typing.Sequence[typing.Callable] = None,
        lifespan: typing.Callable[["Starlette"], typing.AsyncGenerator] = None,
    ) -> None:
        # The lifespan context function is a newer style that replaces
        # on_startup / on_shutdown handlers. Use one or the other, not both.
        assert lifespan is None or (
            on_startup is None and on_shutdown is None
        ), "Use either 'lifespan' or 'on_startup'/'on_shutdown', not both."

        self._debug = debug
        self.state = State()
        self.router = Router(
            routes, on_startup=on_startup, on_shutdown=on_shutdown, lifespan=lifespan
        )
        self.exception_handlers = (
            {} if exception_handlers is None else dict(exception_handlers)
        )
        self.user_middleware = [] if middleware is None else list(middleware)\
        ...

    async def __call__(self, scope: Scope, receive: Receive, send: Send) -> None:
        scope["app"] = self
```

`__call__` 方法将 `Starlette` 类变成了一个可调用的`应用程序`


### `中间件`

`fastapi` 给出了使用中间件的示例

```python
from starlette.applications import Starlette
from starlette.middleware import Middleware
from starlette.middleware.base import BaseHTTPMiddleware


class CustomMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request, call_next):
        response = await call_next(request)
        response.headers["Custom-Header"] = "Example"
        return response


app = Starlette()
app.add_middleware(CustomMiddleware)
```

看一下 `BaseHTTPMiddleware` 类的实现

```python
import asyncio
import typing

from starlette.requests import Request
from starlette.responses import Response, StreamingResponse
from starlette.types import ASGIApp, Message, Receive, Scope, Send

RequestResponseEndpoint = typing.Callable[[Request], typing.Awaitable[Response]]
DispatchFunction = typing.Callable[
    [Request, RequestResponseEndpoint], typing.Awaitable[Response]
]


class BaseHTTPMiddleware:
    def __init__(self, app: ASGIApp, dispatch: DispatchFunction = None) -> None:
        self.app = app
        self.dispatch_func = self.dispatch if dispatch is None else dispatch

    async def __call__(self, scope: Scope, receive: Receive, send: Send) -> None:
        if scope["type"] != "http":
            await self.app(scope, receive, send)
            return

        request = Request(scope, receive=receive)
        response = await self.dispatch_func(request, self.call_next)
        await response(scope, receive, send)

    ...
```

`CustomMiddleware` 类派生于 `BaseHTTPMiddleware` 。



