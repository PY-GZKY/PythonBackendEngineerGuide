# 如何搭建一台后端服务器

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

FastAPI 提供了内置的 Swagger UI 文档生成器，使得生成和查看交互式 API 文档变得非常简单。

Swagger 是一个用于描述和可视化 RESTful API 的开源框架，它可以帮助开发人员和用户更好地理解和使用 API。

当使用 FastAPI 构建 API 时，它会自动根据代码中的注释和类型提示生成 Swagger 文档。通过访问 API 的根端点（例如，http://localhost:8000/docs），您可以在浏览器中查看交互式的 Swagger UI 文档。这个文档包含了 API 的端点、请求方法、请求参数、响应示例等详细信息，使得用户可以方便地测试和理解 API 的功能和使用方式。

FastAPI 的 Swagger UI 文档不仅提供了清晰的接口描述，还支持请求参数的自动验证、响应模型的展示、请求示例的生成等功能，使得与 API 的交互更加方便和直观。

总的来说，FastAPI 的 Swagger UI 文档生成器是一个强大且实用的工具，可以帮助开发人员更好地展示和共享他们构建的 API。

在 `Java` 或者 `Go` 中我们可能需要通过注解来构建 `Swagger API`文档，但是在这里你完全不用担心这一步。


