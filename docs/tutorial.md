# 快速开始：
首先，你可以创建一个`app.py`的文件

然后写上：

```
from jolla import WebApp,jolla_server,render

def index(request):
    return render('index.html')

class app(WebApp):
    urls=[
        (r'/',index)
    ]

if __name__=="__main__":
    server=jolla_server(app)
    server.run_server()
```

然后呢，在同目录下，创建`templates`文件夹，并增加一个`index.html`，项目路径基本是这样的：

```
.
├── app.py
└── templates
    └── index.html
```

接着，在 `index.html` 里面写上：

```
<html>

<head>
  <title>Index</title>
</head>

<body>
  <h1>Hello world</h1>
</body>

</html>
```

接着，运行：

```
python app.py
```

如果你有`httpie`，可以在终端输入 `http http://127.0.0.1:8000` 可以看到：

```

HTTP/1.1 200 OK
Content-Length: 94
Content-Type: text/html
Date: Fri, 06 May 2016 12:02:20 GMT
Server: Jolla/1.0

<html>

<head>
  <title>Index</title>
</head>

<body>
  <h1>Hello world</h1>
</body>

</html>
```

最简单的程序就出来了。

--------------------------------------------------------------------------------

## JSON返回
但那只是最简单的render出一个网页

而`Jolla` 最主要是做一个 API server,所以我们把下面的语句加入 `app.py`:

```
from jolla import WebApp, jolla_server, render, render_json


def index(request):
    return render('index.html')


def status(request):
    aljun = {'name': 'aljun', 'education': {
        'college': 'BUCT', 'subject': 'Chemistry'}, 'age': '20'}
    return render_json(aljun)


class app(WebApp):
    urls = [
        (r'/', index),
        (r'/status', status)
    ]

if __name__ == "__main__":
    server = jolla_server(app)
    server.run_server()
```

同样的，我们在终端输入 `http http://127.0.0.1:8000/status`

可以看到：

```
HTTP/1.1 200 OK
Content-Length: 95
Content-Type: application/json
Date: Fri, 06 May 2016 12:10:36 GMT
Server: Jolla/1.0

{
    "age": "20",
    "education": {
        "college": "BUCT",
        "subject": "Chemistry"
    },
    "name": "aljun"
}
```

--------------------------------------------------------------------------------

## 路由系统
可以看到，我们之前的路由都写死了，怪难受的，我们试着这样写 `r/user/<id>`，这样我们就能获得诸如 `/user/1` 或是 `/user/aljun`这样的路由了，而它们都在 `request`参数里面，根据你写的 `<` 和 `>`中间的字符串，`Jolla`会加入在`request`里，你可以通过 `request['id']` 得到这个值。

来，我们试试，把`app.py`改成：

```

from jolla import WebApp, jolla_server, render, render_json


def index(request):
    return render('index.html')


def status(request):
    aljun = {'name': 'aljun', 'education': {
        'college': 'BUCT', 'subject': 'Chemistry'}, 'age': '20'}
    return render_json(aljun)


def user(request):
    user = request['id']
    return render_json({'user': user})


class app(WebApp):
    urls = [
        (r'/', index),
        (r'/status', status),
        (r'/user/<id>', user)
    ]

if __name__ == "__main__":
    server = jolla_server(app)
    server.run_server()
```

同样的，我们跑 `http http://127.0.0.1:8000/user/aljun` ，就能看到：

```
HTTP/1.1 200 OK
Content-Length: 19
Content-Type: application/json
Date: Fri, 06 May 2016 12:25:06 GMT
Server: Jolla/1.0

{
    "user": "aljun"
}
```

这样我们的路有自由性就很强了哈＝ ＝＋

--------------------------------------------------------------------------------

## HTTP动词
可是HTTP有很多动词呀，而且还要传值的，比如有人输入表单登陆什么的，`Jolla`也是很好的支持，这些参数都被加在了 `request`参数里面，我们可以这么写：

```

from jolla import WebApp, jolla_server, render, render_json


def index(request):
    return render('index.html')


def status(request):
    aljun = {'name': 'aljun', 'education': {
        'college': 'BUCT', 'subject': 'Chemistry'}, 'age': '20'}
    return render_json(aljun)


def user(request):
    user = request['id']
    return render_json({'user': user})


def post(request):
    if request['method'] == 'GET':
        return render_json({'method': 'GET', 'data': request['data']})
    if request['method'] == 'POST':
        return render_json({'method': 'POST', 'data': request['data']})


class app(WebApp):
    urls = [
        (r'/', index),
        (r'/status', status),
        (r'/user/<id>', user),
        (r'/post', post)
    ]

if __name__ == "__main__":
    server = jolla_server(app)
    server.run_server()
```

然后呢，你使用 python的 `request`传值试试：

```

In [1]: import requests

In [2]: r=requests.get('http://127.0.0.1:8000/post',data={'name':'Jolla'})

In [3]: r.text
Out[3]: u'{\n"data": {\n"name": "Jolla"\n}, \n"method": "GET"\n}'

In [4]: r=requests.post('http://127.0.0.1:8000/post',data={'name':'Jolla','qqq':'www'})

In [5]: r.text
Out[5]: u'{\n"data": {\n"qqq": "www", \n"name": "Jolla"\n}, \n"method": "POST"\n}'
```

可以看到，你传的值是在 `request['data']`里面，以字典形式存在的。

--------------------------------------------------------------------------------

## Request参数
然后我们来看看细节，那么request里面到底都包括什么呢？

我们修改 `app.py`

```

from jolla import WebApp, jolla_server, render, render_json


def index(request):
    return render('index.html')


def status(request):
    aljun = {'name': 'aljun', 'education': {
        'college': 'BUCT', 'subject': 'Chemistry'}, 'age': '20'}
    return render_json(aljun)


def user(request):
    user = request['id']
    return render_json({'user': user})


def post(request):
    if request['method'] == 'GET':
        return render_json({'method': 'GET', 'data': request['data']})
    if request['method'] == 'POST':
        return render_json({'method': 'POST', 'data': request['data']})

def detail(request):
    return render_json({'request':request})


class app(WebApp):
    urls = [
        (r'/', index),
        (r'/status', status),
        (r'/user/<id>', user),
        (r'/post', post),
        (r'/detail',detail)
    ]

if __name__ == "__main__":
    server = jolla_server(app)
    server.run_server()
```

然后，我们执行： `http http://127.0.0.1:8000/detail` 在终端，就可以看到：

```

HTTP/1.1 200 OK
Content-Length: 289
Content-Type: application/json
Date: Fri, 06 May 2016 12:45:45 GMT
Server: Jolla/1.0

{
    "request": {
        "content_length": null,
        "content_type": null,
        "cookies": null,
        "data": {},
        "http_accept_encoding": null,
        "http_connect": "keep-alive",
        "http_port": "127.0.0.1:8000",
        "http_protocol": "HTTP/1.1",
        "method": "GET",
        "query_string": {},
        "user_agent": "HTTPie/0.9.3"
    }
}
```

即，`request`其实是包括了很多的环境变量，方便我们使用。而且是`Jolla`考虑到有些粗心的人的存在（如我），你即使不添加 `request`在你的函数里面，都是可以的。

--------------------------------------------------------------------------------

## 增加Header
我们都知道，header的用处很多，所以我们想改改，其实`Jolla`提供了增加的办法：

我们再改改我们的`app.py`:

```

from jolla import WebApp, jolla_server, render, render_json


def index(request):
    return render('index.html')


def status(request):
    aljun = {'name': 'aljun', 'education': {
        'college': 'BUCT', 'subject': 'Chemistry'}, 'age': '20'}
    return render_json(aljun)


def user(request):
    user = request['id']
    return render_json({'user': user})


def post(request):
    if request['method'] == 'GET':
        return render_json({'method': 'GET', 'data': request['data']})
    if request['method'] == 'POST':
        return render_json({'method': 'POST', 'data': request['data']})

def detail(request):
    return render_json({'request':request})

def header(request):
    return render('index.html',extra_header=[('Vary','Accept-Encoding'),('X-Powered-By','PHP 5.4.28')])


class app(WebApp):
    urls = [
        (r'/', index),
        (r'/status', status),
        (r'/user/<id>', user),
        (r'/post', post),
        (r'/detail',detail),
        (r'/header',header)
    ]

if __name__ == "__main__":
    server = jolla_server(app)
    server.run_server()
```

然后我们跑 `http http://127.0.0.1:8000/header` ，就可以看到：

```

HTTP/1.1 200 OK
Content-Length: 94
Content-Type: text/html
Date: Fri, 06 May 2016 12:53:58 GMT
Server: Jolla/1.0
Vary: Accept-Encoding
X-Powered-By: PHP 5.4.28

<html>

<head>
  <title>Index</title>
</head>

<body>
  <h1>Hello world</h1>
</body>

</html>
```

可以看到，被我们加上去了。

--------------------------------------------------------------------------------

## Session
这时候，我们希望有个对话能持久化我们数据，我们就需要用到`session`了，这里我还是推荐使用诸如`redis`这种来当你的session，不过`Jolla`有自带的 `session` ，它包括 `empty`,`session_count`,`add_value`,`check_value`,`del_value`,`get_value`这几个api可以调用。

```

def blog(request):
    if request['method']=="POST":
        session.add({'login_username':request['data']['username']})
        return redirect("/")
    if request['method']=='GET':
        if session.check_value(request['data']['username']):
            return redirect("/loged")
        else:
            return redirect("/register")
```

--------------------------------------------------------------------------------

## 部署（logging与host和port）
有时，我们希望能把很多链接打成log用于生产环境下，那么怎么在`jolla`中把log打在一个文件中呢？

只需在`server_run`这个，加上一个参数：

```
if __name__ == "__main__":
    server = jolla_server(app,log="errer.log")
    server.run_server()
```

这样，我们跑我们的程序，之后在`error.log`就可以看到我们的生产log了。

如我们在终端输入： tail error.log

可以看到：

```
[05-26-2016 14:14:55] INFO:127.0.0.1 - - [2016-05-26 14:14:55] "GET / HTTP/1.1" 200 214 0.001587
[05-26-2016 14:14:58] INFO:127.0.0.1 - - [2016-05-26 14:14:58] "GET / HTTP/1.1" 200 214 0.002000
[05-26-2016 14:15:02] WARNING:<REQUEST /qq/ NOT FOUND IN ROUTE CONFIGURATION>
[05-26-2016 14:15:02] INFO:127.0.0.1 - - [2016-05-26 14:15:02] "GET /qq HTTP/1.1" 404 140 0.000875
```

对于部署，我们肯定不是在本地的`localhost`运行，这个也很简单，在`jolla`的`server`里面内置了个`listener`，可以把`host`和`port`给他就可以了

```
if __name__ == "__main__":
    server = jolla_server(app,log="errer.log",listener=("xx.xx.xx.xx",80))
    server.run_server()
```

这样，我们的`jolla server`就运行在了`xx.xx.xx.xx:80`上面了

--------------------------------------------------------------------------------

## 多进程
现在基本都是多核处理器，能好好的使用好多核的性能的人才是王者，`jolla`当然也加入了多进程的使用方法，使用`python`内置的`multiprocessing`做多进程

> 这里提醒一下，跑的进程数最好是自己电脑的核数

```

from jolla import SessionError,HTTP404Error,session,plugins,plugins
from gevent.server import _tcp_listener
from multiprocessing import Process, current_process, cpu_count

session = session()


def index(request):

    return plugins.render('indexwww.html',extra_header=[('Vary','Accept-Encoding'),('X-Powered-By','PHP 5.4.28')])

def say(request):

    return plugins.render_json({'qqq':{'www':'哈哈哈'}},indent=4)


class app(server.WebApp):
    urls = [
        (r'/', index),
        (r'/data', say)
    ]
listener = _tcp_listener(('127.0.0.1', 8001))

def serve_forever(listener):
    server.jolla_server(listener=listener, app=app).run_server()

if __name__ == '__main__':
    number_of_processes = 4
    print 'Starting %s processes' % number_of_processes
    for i in range(number_of_processes):
        Process(target=serve_forever, args=(listener,)).start()
```

这样，就能很好的使用多进程了，我在我的mac上测试，用了3核效率是单核的2倍左右。

好吧，差不多就到这里，希望你们喜欢。
