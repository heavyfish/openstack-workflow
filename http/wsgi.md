---
description: WSGI 的全称是：Web Server Gateway Interface，介于 web server 和应用 server 之间的接口规范。
---

# WSGI

## 1、WSGI是什么

**WSGI：**全称是`Web Server Gateway Interface`，`WSGI`不是服务器，`python`模块，框架，`API`或者任何软件，只是一种规范，描述`web server`如何与`web application`通信的规范。`server`和`application`的规范在[PEP 3333](https://link.jianshu.com/?t=https://www.python.org/dev/peps/pep-3333/)中有具体描述。要实现WSGI协议，必须同时实现web server和web application，当前运行在`WSGI`协议之上的`web`框架有`Bottle`, `Flask`, `Django`。

但是， WSGI 只是一个规范，只有web  server和框架的作者们和维护者们实际去实现 WSGI 规范后，我们才会这个server和框架使用了wsgi协议。

## 2、规范概述

WSGI interface有两端`【server / gateway】` 和 `【application / framework】`

server 端调用 application 端提供的**可调用** object，该 object 的提供方式取决于 server/gateway 。此处假定某些 server 或 gateway 将需要 application 的部署者编写一个简短的脚本来创建 server/gateway 的 instance，并为其提供 application object。 还有一些  server/gateway 可以使用配置文件或其他机制来指定应从何处导入或以其他方式获取application object。

除了纯粹的 server 和 application，还可以创建同时实现此两种规范的 "middleware"。在被server 调用时，它是 application；在它调用 application 时，它是server。

”**可调用**“ 表示具有 \_\_call\_\_ 方法的 函数、方法、类或实例

![](../.gitbook/assets/image%20%2852%29.png)

参见上图【application &lt;-&gt; middleware】间、【server &lt;-&gt; middleware】间通信时，就用了WSGI协议。

## 3、字符串类型

通常，HTTP处理 bytes，这意味着此规范主要是关于处理 bytes 的。

但是，这些 bytes 的内容通常具有某种文本解释，在Python中，string 是处理文本的最便捷方法。

在python2中，str 是 bytes类型，在python3中，str 是 Unicode类型。因此需要在API的可用性和HTTP上下文中的bytes和文本之间正确转换之间保持谨慎的平衡，尤其是要支持在具有不同str类型的Python实现之间移植代码。

因此，WSGI定义了两种“string”：

* "Native" strings \(通常使用名为str的类型实现\) 用于 请求/响应 header 和元数据
* "Bytestrings" \(使用Python 3中的 bytes 实现，在其他地方使用str\), 用于请求和响应的主体（例如，POST / PUT输入数据和HTML页面输出）。

不要弄混了：即使 Python 的 str 实际上是 unicode ，native strings 的内容也必须支持通过 Latin-1转换成bytes

一句话：当你在PEP 3333中看到 string 时，它代表『原生』字符串，例如一个 str 类型的对象，不管它内部实现上用的是字节还是 unicode。当你看到 bytestring，应该视作一个在 python3中的bytes对象， 在 python2中的 str 对象

所以，尽管在某种意义上，http就是很像字节，有多个方便的API可以将其转换成 Python 的默认 str 类型。

## 4、Application/Framework端

application object 只是一个接受两个参数的可调用对象。 术语“object”不应误解为需要实际的object instance：函数，方法，类或具有\_\_**call\_\_**方法的实例都可以用作application object。 application object 必须能够被多次调用，因为几乎所有servers/gateway（CGI除外）都会发出此类重复请求。

示例如下：一个函数，一个类

```text
HELLO_WORLD = b"Hello world!\n"

def simple_app(environ, start_response):
    """Simplest possible application object"""
    status = '200 OK'
    response_headers = [('Content-type', 'text/plain')]
    start_response(status, response_headers)
    return [HELLO_WORLD]

class AppClass:
    """Produce the same output, but using a class

    (Note: 'AppClass' is the "application" here, so calling it
    returns an instance of 'AppClass', which is then the iterable
    return value of the "application callable" as required by
    the spec.

    If we wanted to use *instances* of 'AppClass' as application
    objects instead, we would have to implement a '__call__'
    method, which would be invoked to execute the application,
    and we would need to create an instance for use by the
    server or gateway.
    """

    def __init__(self, environ, start_response):
        self.environ = environ
        self.start = start_response

    def __iter__(self):
        status = '200 OK'
        response_headers = [('Content-type', 'text/plain')]
        self.start(status, response_headers)
        yield HELLO_WORLD
```

## 5、Server/Gateway 端

server端代码略过，此处以一个简单示例为例：

此处此函数实现了 application

```text
# /bin/python
from wsgiref.simple_server import make_server


def application(environ, start_response):
   start_response('200 OK', [('Content-Type', 'text/html')])
   body = r'<h1>Hello,%s!</h1>'%(environ['PATH_INFO'][1:] or 'web')
   return [body.encode('utf-8')]

#class App(object):
#    def __init__(self):
#        pass
#
#    def __call__(self, environ, start_response):
#        start_response('200 OK', [('Content-Type', 'text/html')])
#        self.body = r'<h1>Hello,%s!</h1>' % (environ['PATH_INFO'][1:] or 'web')
#        return [self.body.encode('utf-8')]

#class App(object):
#    def __init__(self,environ, start_response):
#        return self.app(environ, start_response)
#
#    def app(self, environ, start_response):
#        start_response('200 OK', [('Content-Type', 'text/html')])
#        self.body = r'<h1>Hello,%s!</h1>' % (environ['PATH_INFO'][1:] or 'web')
#        return [self.body.encode('utf-8')]



if __name__ == '__main__':
    httpd = make_server('127.0.0.1', 8000, application)
    print('Serving HTTP on port 8000...')
    httpd.serve_forever()
```

## 6、规定细节

application object 必须接受两个位置参数。 为了说明起见，我们将它们命名为environ和start\_response，但是不需要它们一定叫此名称。 servers必须使用位置（而非关键字）参数调用application object。 （例如，result = application（environ，start\_response））。

environ参数是一个字典对象，包含CGI样式的环境变量。 该对象必须是内置的Python字典（而不是子类，UserDict或其他字典仿制），并且允许application 以所需的任何方式修改字典。 该字典还必须包含某些WSGI必需的变量（在后面的部分中进行描述），并且还可以包含server 特定的扩展变量，这些扩展变量是根据以下将要描述的约定命名的。

start\_response参数是可调用的，接受两个必需的位置参数和一个可选参数。 为了便于说明，我们已将这些参数命名为status，response\_headers和exc\_info，但并不需要它们具有这些名称，并且应用程序必须使用位置参数（例如start\_response（status，response\_headers））调用start\_response可调用对象。

status参数是形式为“ 999 Message here”的状态字符串，response\_headers是描述HTTP响应header 的`（header_name, header_value）`元组的列表。 可选的exc\_info参数在以下 [The start\_response\(\) Callable](https://www.python.org/dev/peps/pep-3333/#the-start-response-callable) and [Error Handling](https://www.python.org/dev/peps/pep-3333/#error-handling)部分中进行了描述。 仅当应用程序捕获了错误并试图向浏览器显示错误消息时，才使用它。

...后续暂且忽略。

## 7、总结

从上所述，我们应对WSGI 协议 有一种模糊的感觉，WSGI规定了server端和application端关于沟通、处理、错误处理等各方便的细节

WSGI协议规定了：

> 1、HTPP的bytes应如何处理转换
>
> 2、applicaton object 的基本规范
>
> 3、HTTP Header的处理
>
> ......

## 8、参考文档

[WSGI-PEP 3333](https://www.python.org/dev/peps/pep-3333/#preface-for-readers-of-pep-333)

[WSGI规范\(PEP 3333\) 第一部分](https://zhuanlan.zhihu.com/p/27600327)



