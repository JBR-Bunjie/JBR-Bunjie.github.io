## Flask如何处理请求

### 一个简单但完整的示例

```python
from flask import Flask, jsonify

app = Flask(__name__)


@app.route('/api')
def my_microservice():
    return jsonify({'Hello': 'World!'})


if __name__ == '__main__':
    app.run()
```

此时，当我们访问`/api`时，应用会返回一个JSON映射。

1. 变量\_\_name\_\_

> 变量\_\_name\_\_是这个应用软件包的名称，而当运行一个单独的Python模块时，变量\_\_name\_\_会赋值为\_\_main\_\_
>
> Flask会使用这个变量实例化一个新的日志日志记录器\(logger\)，并在磁盘上定位这个模块所在文件的路径。
>
> Flask将使用该文件的目录作为助手程序的根目录(例如与应用程序相关的配置文件)，并根据此目录确定静态文件目录(static)与模板目录(templates)的默认存放位置

在shell中运行当前模块时，Flask会运行其中的内置Web服务器，并在默认在5000端口监听传入的请求

2. 访问/api

```shell
root@BUNJIESP8:/mnt/c/Users/m1518/Project/flask# curl -v http://127.0.0.1:5000/api
*   Trying 127.0.0.1:5000...
* Connected to 127.0.0.1 (127.0.0.1) port 5000 (#0)
> GET /api HTTP/1.1
> Host: 127.0.0.1:5000
> User-Agent: curl/7.81.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< Server: Werkzeug/2.2.2 Python/3.10.4
< Date: Sat, 10 Sep 2022 11:43:02 GMT
< Content-Type: application/json
< Content-Length: 19
< Connection: close
< 
{"Hello":"World!"}
* Closing connection 0
```

可以发现，我们得到了一个合法的JSON响应和正确的消息头。

3. jsonity函数

该函数会将Python字典类型转换为合法的JSON响应，并在添加适当的Content-Type消息头后，将映射信息存储到响应体中



### request对象

与大多数Web框架不同，flask不需要显示地将request对象传递到代码中——它隐式地提供了一个全局的request变量，并用该全局的变量来指向当前的request对象。Flask把传入的HTTP请求解析为WSGI环境字典，并利用它来创建这个对象

这样：当服务器的响应不依赖请求的内容时，就没必要处理它。视图只需要确保返回了客户端应该获取的内容，并确保内容能够被Flask序列化即可

### 了解底层到底发生了什么

增加print方法，了解curl访问具体过程

```python
from flask import Flask, jsonify, request

app = Flask(__name__)


@app.route('/api')
def my_microservice():
    print(request)
    print(request.environ)
    response = jsonify({'Hello': 'World!'})
    print(response)
    print(response.data)
    return response


if __name__ == '__main__':
    print(app.url_map)
    app.run()
```

有：

```shell
(venv) root@BUNJIESP8:/mnt/c/Users/m1518/Project/flask/chapter2# python app.py 
Map([<Rule '/static/<filename>' (OPTIONS, GET, HEAD) -> static>,
 <Rule '/api' (OPTIONS, GET, HEAD) -> my_microservice>])
 * Serving Flask app 'app'
 * Debug mode: off
WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
 * Running on http://127.0.0.1:5000
Press CTRL+C to quit
<Request 'http://127.0.0.1:5000/api' [GET]>
{'wsgi.version': (1, 0), 'wsgi.url_scheme': 'http', 'wsgi.input': <_io.BufferedReader name=4>, 'wsgi.errors': <_io.TextIOWrapper name='<stderr>' mode='w' encoding='utf-8'>, 'wsgi.multithread': True, 'wsgi.multiprocess': False, 'wsgi.run_once': False, 'werkzeug.socket': <socket.socket fd=4, family=AddressFamily.AF_INET, type=SocketKind.SOCK_STREAM, proto=0, laddr=('127.0.0.1', 5000), raddr=('127.0.0.1', 54614)>, 'SERVER_SOFTWARE': 'Werkzeug/2.2.2', 'REQUEST_METHOD': 'GET', 'SCRIPT_NAME': '', 'PATH_INFO': '/api', 'QUERY_STRING': '', 'REQUEST_URI': '/api', 'RAW_URI': '/api', 'REMOTE_ADDR': '127.0.0.1', 'REMOTE_PORT': 54614, 'SERVER_NAME': '127.0.0.1', 'SERVER_PORT': '5000', 'SERVER_PROTOCOL': 'HTTP/1.1', 'HTTP_HOST': '127.0.0.1:5000', 'HTTP_USER_AGENT': 'curl/7.81.0', 'HTTP_ACCEPT': '*/*', 'werkzeug.request': <Request 'http://127.0.0.1:5000/api' [GET]>}
<Response 19 bytes [200 OK]>
b'{"Hello":"World!"}\n'
127.0.0.1 - - [10/Sep/2022 22:55:47] "GET /api HTTP/1.1" 200 -
```

#### 路由匹配

路由匹配发生在app.url_map中，这是Werzeug中Map类的一个实例。`Map([<Rule '/static/<filename>' (OPTIONS, GET, HEAD) -> static>, <Rule '/api' (OPTIONS, GET, HEAD) -> my_microservice>])`

该类使用正则表达式来判定被`@app.route`装饰的函数时候与传入的请求匹配，路由匹配只会姜茶route调用里的路径参数来判断函数时候匹配客户端的请求，默认情况下，声明式路由只支持`GET`, `OPTIONS`, `HEAD`方法的调用，如果使用了不支持的HTTP方法，则会出现405Method Not Allowed响应，并在Allow响应头中返回其所支持的HTTP方法列表（正常访问不会出现Allow响应头）

当需要支持指定HTTP请求方式时，我们需要给装饰器增加额外的参数：

```python
@app.route('/api', method=['POST', 'DELETE', 'GET'])
```



不过，

##### 变量与转换器

路由系统支持变量如：`/person/<person_id>`







#### 请求



#### 响应



## Flask的内置特性















## 微服务骨架







拓展：

**这个 __name__ 变量可能取什么值？**

当你直接执行一段脚本的时候，这段脚本的 **__name__**变量等于 **'__main__'**，当这段脚本被导入其他程序的时候，**__name__** 变量等于脚本本身的名字。





**这个 __name__ 拿来做什么的？**

作为 Python 的内置变量，**__name__**变量（前后各有两个下划线）还是挺特殊的。它是每个 Python 模块必备的属性，但它的值取决于你是如何执行这段代码的。

在许多情况下，你的代码不可能全部都放在同一个文件里，或者你在这个文件里写的函数，在其他地方也可以用到。为了更高效地重用这些代码，你需要在 Python 程序中导入来自其他文件的代码。

所以，在**__name__** 变量的帮助下，你可以判断出这时代码是被直接运行，还是被导入到其他程序中去了。

[Python 的 __name__ 变量，到底是个什么东西？ - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/57309137)

