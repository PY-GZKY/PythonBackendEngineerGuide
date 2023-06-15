
> https://fastapi.tiangolo.com

> https://github.com/tiangolo/fastapi

## 安装

```shell
pip install fastapi
pip install uvicorn
```

`uvicorn` 是一个高性能的 `ASGI` 服务器，后面会讲到。

## 构建一个简单的应用程序

新建一个 main.py 文件并编写一下代码

```python
import uvicorn
from fastapi import FastAPI

app = FastAPI()

@app.get("/pi")
def home(name="pi"):
    return f"Hello! {name}"

if __name__ == "__main__":
    uvicorn.run(app, host='127.0.0.1', port=8000, debug=True)
```

> 可以看到 `fastapi` 的语法和 `flask`非常接近，不管是从设计模式还是感官上来说都有着异曲同工之妙 ！！

我们通过:
````shell
python main.py
````

这样一个简单的`fastapi` 应用就起来了并且使用了 `uvicron` 服务器启动，输出如下:

```shell
INFO: Uvicorn running on http://0.0.0.0:8000 (Press CTRL+C to quit)
INFO: Started reloader process [37928] using statreload
INFO: Started server process [41144]
INFO: Waiting for application startup.
INFO: Application startup complete.
```

在开发过程中，我们想知道代码被改变时接口或页面上实时的同步更改，指定 `--reload `参数为 `True` 可以解决这个问题，也叫热加载 ！！

```shell
if __name__ == "__main__":
    uvicorn.run(app='main:app', host="0.0.0.0", port=8000,reload=True, debug=True)
```

## `Swagger UI`

`FastApi` 提供了优美的 `Swagger` 交互式文档。

在 `Java` 或者 `Go` 中我们可能需要通过注解来构建 `Swagger API`文档，但是在这里你完全不用担心这一步。





