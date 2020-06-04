---
description: apache、nginx、wsgi、uwsgi、uWSGI
---

# 名词解析

## 1、基本概念

**1. wsgi: \(the Python Web Server GateWay Interface\) 是一种通信协议**

> *  `python web框架编写的应用程序` &lt;==&gt; `后端服务器`之间的通信规范.
> *  [PEP 3333](https://www.python.org/dev/peps/pep-3333/)中描述了`web server`如何与`web application`通信的规范
> * 要实现wsgi协议，必须同时实现`web server` 和`web application`
> * 当前运行在`WSGI`协议之上的`web`框架有`Bottle`, `Flask`, `Django`

**2. uwsgi: 是一种线路协议**

> * 与wsgi是两种东西，是`uWSGI服务器`的独占协议

**3. uWSGI: 是一个web服务器的名字**

> * uWSGI旨在为部署分布式集群的网络应用开发一套完整的解决方案
> * 主要面向web及其标准服务。由于其`可扩展性`，能够被无限制的扩展用来支持更`多平台`和`多语言`
> * 实现了`WSGI协议`、`uwsgi协议`等
> *  `uWSGI服务器`自己实现了`基于uwsgi协议的server部分`，我们只需要在uwsgi的配置文件中`指定application的地址`，uWSGI就能直接和`应用框架中的WSGI application`通信。
> * **uWSGI的主要特点是：**
>
> > * 超快的`性能`
> > *  `低内存`占用
> > *  `多app`管理
> > * 详尽的`日志功能`（可以用来分析app的性能和瓶颈）
> > *  `高度可定制`（内存大小限制，服务一定次数后重启等）

**4. Apache: 世界使用排名第一的Web服务器**

**5. Nginx: 轻量级的Web服务器/反向代理服务器及电子邮件（IMAP/POP3）代理服务器**



