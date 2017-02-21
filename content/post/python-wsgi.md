+++
menu = ""
categories = [
    "backend"
]
description = ""
banner = ""
tags = [
    "Python"
]
date = "2017-02-21T09:15:24+08:00"
title = "Python Web Server Gateway Interface"
images = [
]
draft = false

+++


# 引言

在Python的Web开发界，有着多种多样的框架，比如Flask, Django, Falcon等等，可谓是百花齐放。许多新入门的同学，都会纠结使用哪个框架比较好。在这里 我们不去回答这个问题，而是去寻找它们的共通点，也就是本篇的主题：

    Web Server Gateway Interface (WSGI, 可以分开来读, 也可以读作'wiz-gee')

# WSGI是什么

WSGI不是一个框架，也不是一个Python的库，也不是服务，API或者其他任何类型的软件。它只是服务(`Web Server`)和应用(`Web Application`)之间的一种简单通用的接口规范。它定义了服务和应用两端，除此之外还有一种叫中间件(`Middleware`)的组件，它同时实现了服务端和应用端。

> 如果一个应用满足了WSGI，那么它就能在所有满足WSGI的服务上跑起来。

### Application

`Application`处理请求，并返回处理的结果。它是一个可调用(`callable`)的对象，可以是一个函数，方法，实现了`__call__`或者`__iter__`方法的类。

#### 函数
```python
def app(environ, start_response):
    start_response('200 OK', [('Content-type', 'text/plain')])
    return ['Hello world!']
```

<!--more-->

#### 方法
```python
class Demo(object):
    def __init__(self):
        # do something
        ...
    
    def app(self, environ, start_response):
        start_response('200 OK', [('Content-type', 'text/plain')])
        return ['Hello world!']
```

#### \_\_call__类
```python
class Application(object):
    def __init__(self):
        # do something
        ...
        
    def __call__(self, environ, start_response):
        start_response('200 OK', [('Content-type', 'text/plain')])
        return ['Hello world!']
```

#### \_\_iter__类

```python
class Application(object):
    def __init__(self, environ, start_response):
        self.environ = environ
        self.start_response = start_response
        
    def __iter__(self):
        self.start_response('200 OK', [('Content-type', 'text/plain')])
        yield 'Hello world!'
```

### Server

`Server`接收网络请求，调用`Application`去处理请求，并把其返回的结果传输给客户端。

```python
import os, sys

def run_with_cgi(application):

    environ = dict(os.environ.items())
    environ['wsgi.input']        = sys.stdin
    environ['wsgi.errors']       = sys.stderr
    environ['wsgi.version']      = (1, 0)
    environ['wsgi.multithread']  = False
    environ['wsgi.multiprocess'] = True
    environ['wsgi.run_once']     = True

    if environ.get('HTTPS', 'off') in ('on', '1'):
        environ['wsgi.url_scheme'] = 'https'
    else:
        environ['wsgi.url_scheme'] = 'http'

    headers_set = []
    headers_sent = []

    def write(data):
        if not headers_set:
             raise AssertionError("write() before start_response()")

        elif not headers_sent:
             # Before the first output, send the stored headers
             status, response_headers = headers_sent[:] = headers_set
             sys.stdout.write('Status: %s\r\n' % status)
             for header in response_headers:
                 sys.stdout.write('%s: %s\r\n' % header)
             sys.stdout.write('\r\n')

        sys.stdout.write(data)
        sys.stdout.flush()

    def start_response(status, response_headers, exc_info=None):
        if exc_info:
            try:
                if headers_sent:
                    # Re-raise original exception if headers sent
                    raise exc_info[0], exc_info[1], exc_info[2]
            finally:
                exc_info = None     # avoid dangling circular ref
        elif headers_set:
            raise AssertionError("Headers already set!")

        headers_set[:] = [status, response_headers]
        return write

    result = application(environ, start_response)
    try:
        for data in result:
            if data:    # don't send headers until body appears
                write(data)
        if not headers_sent:
            write('')   # send headers now if body was empty
    finally:
        if hasattr(result, 'close'):
            result.close()
```

### Middleware

`Middleware`同时实现了`Server`和`Application`两端的功能，它接收上层`Server`或者`Middleware`传过来的请求，调用下层`Application`或者`Middleware`来处理，处理的结果又返回给上层。在实际场景中，我们会有多个中间件，这样就形成了中间件栈。

```python
def app(environ, start_response):
    start_response('200 OK', [('Content-type', 'text/plain')])
    return ['Hello world!']
    
class Middleware1(object):
    def __init__(self, app):
        self.app = app
        
    def __call__(self, environ, start_response)
        print 'Middleware1'
        return self.app(environ, start_response)
        

class Middleware2(object):
    def __init__(self, app):
        self.app = app
        
    def __call__(self, environ, start_response)
        print 'Middleware2'
        return self.app(environ, start_response)
        

app = Middleware1(Middleware2(app))
```

![image](/img/wsgi.png)


> 我们现在常用的框架，如Flask, Django都是应用端的，但是它们提供了简单的服务端运行，这样在开发过程中就可以运行起来。

> 服务端实现有库wsgiref(内置库), werkzeug, gunicorn, gevent, uwsgi等。


# 一步步写个Web

### 运行Hello World

```python
# coding=utf-8

from wsgiref.simple_server import make_server


def app(environ, start_response):
    start_response('200 Ok', [('Content-Type', 'text/plain')])
    return ['Hello World!']


if __name__ == '__main__':
    server = make_server('127.0.0.1', 8080, app)
    print 'Server start at 127.0.0.1:8080...'
    server.serve_forever()

```

### POST方法

`environ`是一个纯字典，它包含了许多WSGI所需要的变量，如请求参数，请求方法等等。

```python
# coding=utf-8

from wsgiref.simple_server import make_server


def app(environ, start_response):
    if environ['REQUEST_METHOD'] == 'POST':
        res = 'Post Request.'
    else:
        res = 'Hello World!'
    start_response('200 Ok', [('Content-Type', 'text/plain')])
    return res


if __name__ == '__main__':
    server = make_server('127.0.0.1', 8080, app)
    print 'Server start at 127.0.0.1:8080...'
    server.serve_forever()
```

### 添加Route

上面的例子中，所有的url请求返回的结果都是一样的。如果我们需要根据不同url返回不同的结果，那么就需要添加路由处理。

```python
# coding=utf-8

from wsgiref.simple_server import make_server


def app(environ, start_response):
    path = environ['PATH_INFO']
    if path == '/':
        res = 'Hello World!'
    elif path == '/test':
        res = 'Get /test'
    else:
        res = 'Cannot handle url.'

    start_response('200 Ok', [('Content-Type', 'text/plain')])
    return res

if __name__ == '__main__':
    server = make_server('127.0.0.1', 8080, app)
    print 'Server start at 127.0.0.1:8080...'
    server.serve_forever()
```

随着我们要处理的url越来越多，app的逻辑会越来越复杂，所以我们用类来重构一下。

```python
# coding=utf-8

from wsgiref.simple_server import make_server


def handle_root():
    return 'Hello World!'


def handle_test():
    return 'Get /test'


class Application(object):

    def __init__(self, routes):
        self.routes = routes

    def __call__(self, environ, start_response):
        handler = self.routes.get(environ['PATH_INFO'])
        if handler is None:
            res = 'Cannot handler url.'
        else:
            res = handler()
        start_response('200 Ok', [('Content-Type', 'text/plain')])
        return res


if __name__ == '__main__':
    routes = {
        '/': handle_root,
        '/test': handle_test
    }
    server = make_server('127.0.0.1', 8080, Application(routes))
    print 'Server start at 127.0.0.1:8080...'
    server.serve_forever()

```

继续重构，为每个url添加method。

```python
# coding=utf-8

from wsgiref.simple_server import make_server


def get_root():
    return 'Hello World!'


def post_root():
    return 'Post root'


def get_test():
    return 'Get /test'


def post_test():
    return 'Post /test'


class Application(object):

    def __init__(self, routes):
        self.routes = routes

    def __call__(self, environ, start_response):
        handler = self.routes.get(environ['PATH_INFO'], {}).get(environ['REQUEST_METHOD'])
        if handler is None:
            res = 'Cannot handler url.'
        else:
            res = handler()
        start_response('200 Ok', [('Content-Type', 'text/plain')])
        return res


if __name__ == '__main__':
    routes = {
        '/': {
            'GET': get_root,
            'POST': post_root
        },
        '/test': {
            'GET': get_test,
            'POST': post_test
        }
    }
    server = make_server('127.0.0.1', 8080, Application(routes))
    print 'Server start at 127.0.0.1:8080...'
    server.serve_forever()
```

### 日志中间件

```python
# coding=utf-8

from wsgiref.simple_server import make_server


def handle_root():
    return 'Hello World!'


def handle_test():
    return 'Get /test'


class Application(object):

    def __init__(self, routes):
        self.routes = routes

    def __call__(self, environ, start_response):
        handler = self.routes.get(environ['PATH_INFO'])
        if handler is None:
            res = 'Cannot handler url.'
        else:
            res = handler()
        start_response('200 Ok', [('Content-Type', 'text/plain')])
        return res


class Middleware(object):

    def __init__(self, app):
        self.app = app

    def __call__(self, environ, start_response):
        print 'Before request.'
        res = self.app(environ, start_response)
        print 'After request.'
        return res


if __name__ == '__main__':
    routes = {
        '/': handle_root,
        '/test': handle_test
    }
    server = make_server('127.0.0.1', 8080, Middleware(Application(routes)))
    print 'Server start at 127.0.0.1:8080...'
    server.serve_forever()

```

# 总结

本篇介绍了WSGI的一些相关概念，然后一步步写了一个简单的web后台。当然，实际开发过程中我们根本不需要这么来做，因为现在的框架已经封装了好了一切。

举个Flask的例子：

```python
# coding=utf-8

from flask import Flask


app = Flask(__name__)


@app.route('/')
def hello():
    return 'Hello World!'


if __name__ == '__main__':
    app.run()
```


# 参考

[PEP333](https://www.python.org/dev/peps/pep-0333/)

[PEP333中文翻译](https://github.com/mainframer/PEP333-zh-CN)