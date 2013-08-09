---
title: 到底什么是 WSGI ?
layout: post
---

# CGI, FCGI, SCGI, WSGI 简介

> CGI: fork-and-excute模式/语言无关
!["cgi"](http://i.imgur.com/bR7r4l.png)

> FCGI: 改良版CGI，为了减轻CGI创建进程的消耗，使用事件驱动或多线程方式处理请求/语言无关 -> [see here](http://www.fastcgi.com/devkit/doc/fcgi-spec.html)
!["fastcgi"](http://i.imgur.com/MdRcKl.png)

fastcgi 进程通过socket接受webserver的请求(使用socket方式可以实现分布式部署)，利用事件驱动或者多线程方式处理请求后返回给web server。而fastcgi则描述了webserver和fastcgi之间的通讯规范，以及fastcgi处理请求的一些规范。

> SCGI(simplecgi) -> [see here](http://python.ca/scgi/protocol.txt)

类似于FCGI，但web server和web app之间的协议更加简单/语言无关

> WSGI -> [see here](http://www.python.org/dev/peps/pep-0333/)
WSGI描述了web server和python app之间的交互规范。

WSGI是更高一个层次的概念，对传统CGI做了一层封装，定义了一个简单通用的接口(标准/协议)用于server和web app/framwork之间的交互。Python语言特有。


# 区别

以上几个 xxGI 都不代表一个可用的实体，也不是一组API或者模块，它们都是server和web app之间交互的规范和协议。

Web Server执行web app的几种方式:

* fork-and-excute

* 嵌入式: mod_wsgi/mod_python(mod_python已过时) + webserver，将python解释器嵌入server，调用web app时由server进程执行app内容，而不用产生子进程。

* 守护进程daemon方式: mod_wsgi/mod_fastcgi + webserver，在server进程之外有独立的守护进程(long-running process)运行着，server将请求丢给这个进程，让这个进程去执行对应的过程，通过WSGI协议和这个进程交互。

也就是说，WSGI既可以以嵌入式方式工作，也可以以daemon方式工作。

虽然工作方式不同，但是这些CGI(这里统称为cgi)并不互相冲突，完全可以协同工作，比如在Django中：
Apache --> mod_fastcgi --> flup(via CGI protocol) --> Django(via WSGI protocol)

*注：flup是python中的实现了WSGI协议的一个模块， [这里](http://wiki.python.org/moin/WSGIImplementations)有其他的实现WSGI协议的模块。*

一些链接：
[how-python-web-frameworks-wsgi-and-cgi-fit-together](http://stackoverflow.com/questions/219110/how-python-web-frameworks-wsgi-and-cgi-fit-together)
[is-there-a-speed-difference-between-wsgi-and-fcgi](http://stackoverflow.com/questions/1747266/is-there-a-speed-difference-between-wsgi-and-fcgi)
[whats-the-difference-between-scgi-and-wsgi](http://stackoverflow.com/questions/257481/whats-the-difference-between-scgi-and-wsgi)

什么是WSGI？

> WSGI is the Web Server Gateway Interface. It is a specification for web servers and application servers to communicate with web applications (though it can also be used for more than that). It is a Python standard, described in detail in PEP 333.

> WSGI是一个提供给WEB服务器和Web App应用程序之间通信的接口。

> A WSGI server (meaning WSGI compliant) only receives the request from the client, pass it to the application and then send the response provided by the application to the client. It does nothing else. All the gory details must be supplied by the application or middleware.

> Middleware are reusable components providing generic services normally handled by frameworks; e.g., a Session object, a Request object, error handling. They're implemented as wrapper functions; Inbound（对内） they can add keys to the dictionary (e.g., quixote.request for a Quixote-style Request object). Outbound（对外） they can modify HTTP headers or translate the body into Latin or Marklar.

WSGI中，分为三个概念： WSGI Server/Gateway <--> Middleware <--> Python application/framwork

WSGI 以回调的方式工作，application/framework 需要提供给Server/Gateway一个可以调用的(callable)对象(函数、类等包含__call__()属性的对象)，Server/Gateway每次收到请求都会调用这个形式的对象:

	application(environ, start_response)

	> @environ: 一个包含类似CGI-Style的环境变量的字典
	> @start_response: 一个接受(status, response_headers, exc_info=None)参数的可调用对象
		> @status: 形如"999 Message here"的状态字符串
		> @response_headers: 一个包含数个(header_name, header_value)的元组

Server/Gateway并不需要知道除了如何调用这个对象之外的任何信息。

结合Django来看：
	在apache上配置一个Django的app( [具体配置](http://code.google.com/p/modwsgi/wiki/IntegrationWithDjango)时，通常需要在apache配置文件中指明一个.wsgi文件，这个.wsgi文件中会包含一个application的函数：

	import django.core.handlers.wsgi
	application = django.core.handlers.wsgi.WSGIHandler()

从django包中找到WSGIHandler()对象，是一个类:

    class WSGIHandler(base.BaseHandler):
		...
		def __call__(self, environ, start_response):
			...
			start_response(status, response_headers)
			return response</pre>

# 关于environ

参数environ中应该包含一些传统的CGI环境变量，以及 **必须** 包含的一些wsgi变量：

* wsgi.version	(1,0) -> 1.0版本

* wsgi.url_scheme

* wsgi.input 

    > 从中可以读出http请求的包体的文件流对象

* wsgi.errors

    > 错误信息输出文件流对象(比如定向到server的错误日志文件)

* wsgi.multithread

    > 如果为True，表明可以将app以线程方式启动。反则不可以。

* wsgi.multiprocess

    > 如果为True，表明将app以进程方式启动。反则不可以。

* wsgi.run_once

    > True说明app在进程生命周期中只能被执行一次。


# 关于start_response(status, response_headers, exc_info=None)

* start_response()对象被调用来产生response，并且必须返回一个可调用的write(body_data)对象

* status: like "200 OK" 的状态码字符串

* response_headers: 包含数个HTTP头部(header_name, header_value)的元组

* Server/Gateway 必须处理好HTTP头部，比如在app返回的response数据中添加一些必要的HTTP头部

* exc_info只有在错误处理时候被添加


其中Middleware中间件提供一些扩展API，或者对传输内容做一些通用的处理工作。

mod_wsgi是apache中的一个模块，这个模块能够支持运行任何提供了WSGI接口规范的python 应用程序，比如Django。 


# 如何结合Django、mod_wsgi、Apache？

官方文档详见 [这里](https://docs.djangoproject.com/en/1.3/howto/deployment/modwsgi/)

* Step1. 安装好的mod_wsgi应该会在apache的安装目录下有modules/mod_wsgi.so库，因此，如果要启动mod_wsgi这个模块，得先在apache配置文件(通常为httpd.conf)中添加:

    LoadModule wsgi_module modules/mod_wsgi.so

* Step2. 在django的项目目录中创建一个wsgi脚本，规范一点就叫django.wsgi

	import os
	import sys
	os.environ\['DJANGO_SETTINGS_MODULE'\]='mysite.settings'

	import django.core.handlers.wsgi
	application = django.core.handlers.wsgi.WSGIHandler()</code></pre>

这个wsgi脚本就是WSGI规范中要求的application/framework提供给Server/Gateway的application接口。

* Step3. 告诉apache这个wsgi接口的位置。在apache的配置文件添加:

*WSGIScriptAlias /mypath /path/to/wsgi/script/django.wsgi*

以及对django.wsgi所在目录的权限设置：

    <Directory /home/coaku/work/django_proj/nickkxu/apache>
        Order allow,deny
        Allow from all
        Require all granted
    </Directory>

需要注意的是，在老的httpd版本中，"Require all granted"这一项可以不写，但在httpd-2.4版本中这一项
必须写，否则访问时会出权限问题。
