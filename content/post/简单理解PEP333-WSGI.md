+++

title = "简单理解PEP333-WSGI"
date = 2018-06-20T14:35:11+08:00
toc = true
tags = ["Python","Web开发"]
categories = ["Web开发"]
banner = "img/avatar.png"
author = "Tacey Wong"

+++

**声明：这篇文章只是为了整体理解WSGI，会忽略很多细节，要详细了解请参看文后的参考资料**

## WSGI概述

WSGI全称是Python Web Server Gateway Interface（Python Web服务网关接口）仅针对名字可以做出一下总结：

+ 1、**Python** Web Server Gateway Interface：这个规范是针对Python的
+ 2、Python **Web Server** Gateway Interface：这个规范是针对Web服务相关的
+ 3、Python Web Server **Gateway** Interface：这个规范是针对服务网关处的
+ 4、Python Web Server Gateway **Interface**：这个规范是一种接口定义，不是具体实现

上面这样按照字面文本切分来理解也许是错的，但是在这里用来梳理理解我认为还是可以的。首先条目1、2、4应该是没问题,比较难理解的是3。3比较难理解是因为在我们的日常学习、开发的经历中，gateway/网关这一个词的准确指代是不明确、不固定的，这个词非常笼统与抽象，业务逻辑上集束处可以被成为网关，协议转换处可以被成为网关，权限分层处可以被称为网关……在基础的HTTP架构里，网关原本是用于称呼协议代理转换的那一部分，比如我们平常通过网页首发QQ邮件，浏览器后服务商后台进行通讯使用的是HTTP（S）协议，可e-mail常规协议是smtp、pop3这些，所以在中间一般经历了HTTP（S）到stmp等协议的转换，这一部分就是传统上基于HTTP服务架构中的网关。那么WSGI中的网关是什么意思呢？是不是协议转换？答案是否定的。在WSGI中所谓的网关是Web服务器与Web应用之间的衔接处，他使用的是类CGI之中网关的意思，就是Web服务器和动态Web程序之间的衔接。静态Web站点需要的软件程序只有Web服务器，Web服务器对文件系统进行相应的映射提供静态资源服务（HTML、图片等），但是动态网站需要处理业务逻辑的程序，业务逻辑程序通过不同的输出渲染输出不同的结果再由Web服务器传送给上游客户端。这样就产生了Web服务器和业务程序衔接的问题，就产生了cgi、fast-cgi。而WSGI是用来统一Web服务器和Python Web应用/框架之间接口的规范。

其实根据上面的描述可以看到业务逻辑程序概念上和网络传输没有关系，比如我们用Flask写的Web程序其实只是业务逻辑，真正处理HTTP网络传输的是Web服务器，只不过Flask为了便于开发自身携带了一个开发型Web服务器。

那WSGI有什么优势呢？为什么要有WSGI呢？首先要说明的是没有WSGI也是可以的，但是如果没有WSGI，每一个Web应用就要有单独的Web服务器，或者对某一个Web服务器做各种的兼容性的改动；有了WSGI规范，那么凡是支持该规范接口的Web服务器和Web应用就可以搭配使用提供服务。


## WSGI规范主要骨架

刨去细节，从整体上看，WSGI对三方的规范：Web服务器规范、Web应用规范、中间件规范

###  Web服务器规范：

> 服务器必须提供两样东西： 一是environ环境字典，二是start_response可调用函数

其中：

+ environ 字典必须包涵一定的数据项，类似于CGI environment
    + environ包含的字段样例如下
    + environ['wsgi.input']        = sys.stdin.buffer
    + environ['wsgi.errors']       = sys.stderr
    + environ['wsgi.version']      = (1, 0)
    + environ['wsgi.url_scheme'] = 'https'
    + ...
+ start_response是接受两个参数的回调函数
    + 参数status： 包涵标准的HTTP状态字符串 例如：200 OK
    + 参数response_headers：标准HTTP响应头列表

服务器像下面这样将包含HTTP请求信息传递给Web应用/框架：
```python
iterable = simple_app(environ, start_response)
for data in iterable:
    pass
    # send data to client
```
可以看出构建响应头、调用start_response并构建数据一可迭代的形式返回是应用/框架的责任,通过HTTP将响应头和数据传递给上游是Web服务器的责任。

###  Web应用/框架规范

> Web应用/框架必须是可调用的，可以是类、函数或者对象， __init__参数、 __call__参数或者函数必须如上面调用的样子

+ 一个environ对象
+ 一个可调用的start_response

> Web应用/框架必须在返回数据之前调用start_response

> Web应用/框架应该以可迭代的形式返回数据，比如：列表

一个简单的WSGI应用的例子

```python
def app(environ, start_response):
    status = '200 OK'
    response_headers = [('Content-type','text/plain')]
    start_response(status, response_headers)
    return ['Hello world!\n']
```

### 中间件规范：

> 中间件组建必须同时遵守Web服务端和Web应用断的规范要求，并添加了一些轻微的限制

> 中间件应尽可能透明

下面是一个简单的将字符串全部转换成大写的中间件
```python
class Upperware:
                def __init__(self, app):
                    self.wrapped_app = app

                def __call__(self, environ, start_response): # 这里遵循应用端规范
                    # 下面两个遵循服务端规范
                    for data in self.wrapped_app(environ, start_response):
                        yield data.upper()
```

## 一个完整的WSGI应用运行样例

```python
# coding:utf-8

"""使用python内带的wsgiref简单服务器作Web服务器"""

from wsgiref.simple_server import make_server
from urlparse import parse_qs


def simple_app(environ, start_response):
    status = '200 OK'
    response_headers = [('Content-type', 'text/plain')]
    start_response(status, response_headers)
    return ['Hello World']


class Upperware:
    def __init__(self, app):
        self.wrapped_app = app

    def __call__(self, environ, start_response):
        for data in self.wrapped_app(environ, start_response):
            yield data.upper()


def main(environ, start_response):
    print environ
    params = parse_qs(environ.get('QUERY_STRING', ''))
    if 'true' in params.get('mid', []):
        app = Upperware(simple_app)
    else:
        app = simple_app
    return app(environ, start_response)


if __name__ == "__main__":
    srv = make_server('localhost', 9797, main)
    srv.serve_forever()

```

如果以mid=true为参数的话访问（不限路径），就会使得中间件生效，返回文本全部大写，若不带该参数或该参数不为‘true’，中间件就不会生效。

再重复提一句：**构建响应头、调用start_response并构建数据一可迭代的形式返回是应用/框架的责任(业务逻辑),通过HTTP将响应头和数据传递给上游是Web服务器的责任（网络传输）。**，Web框架要处理的不是HTTP传输问题，是应用逻辑构建问题，比如说路由系统、上下文、模板渲染等等。还有就是现实世界也不是非此即彼，比如说Web服务器也可以直接构建路由系统，Web服务器和Web框架的编写是技术活，也是力气活（需要实现很多规范的细节）

**声明：这篇文章只是为了简单整体理解WSGI我自己的一点浅薄之见，要详细了解请参看文后的参考资料**

## 参考资料

+ HTTP 权威指南
+ [HTTP MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP)
+ [PEP333](https://legacy.python.org/dev/peps/pep-333/)
+ [PEP3333](https://legacy.python.org/dev/peps/pep-3333/)
+ [Learn about WSGI](https://wsgi.readthedocs.io/en/latest/learn.html)