# PasteDeploy

## 1、概述

Paste Deployment是一个用来发现和配置WSGI应用和服务的系统。对于一个WSGI应用的使用者，可以使用方法loadapp，从一个配置文件或者python egg中，加载出来一个WSGI应用。对于WSGI应用的开发者，它只需要你的应用有一个简单的入口。因此应用的具体实现细节并不会暴露给应用的使用者。

这个设计的结果是，一些系统管理员，在不了解python，或者不了解WSGI应用的详情或者它的容器，也能进行安装和管理。

### 1.1、四种部件

PasteDeploy模型中有4种部件，分别是

![](../.gitbook/assets/image%20%2850%29.png)

## 2、配置文件

一个配置文件有不同的sections。Paste Deploy只关心那些有前缀的sections，例如`[composite:main]` ，其中冒号（:）后面`main`的是这个section的名称，前面的`composite`是这个section的类型。就是那些不符合格式的section **将会被忽略**。

下面是一个典型的配置文件，它将展示如何装载多个应用。

```text
[composite:main]
use = egg:Paste#urlmap
/main/tap = tap
/main/boil/shower = pip_to_shower
 
[app:tap]
paste.app_factory = tap:app_factory
in_arg = water
 
[pipeline:pip_to_shower]
pipeline = boiler shower
 
[filter:boiler]
paste.filter_app_factory = boiler:filter_app_factory
in_arg = water
 
[app:shower]
paste.app_factory = shower:app_factory
in_arg = hot_water
```

### 2.1、composite

```text
[composite:main]
use = egg:Paste#urlmap
/main/tap = tap
/main/boil/shower = pip_to_shower
```

这是一个composite section。代表着它将分发请求给其他的应用。 `use = egg:Paste#urlmap`。代表要用Paste模块里的urlmap去处理分发请求。urlmap是一个比较常用的用于分发请求的应用。它可以根据请求路径的前缀，将请求交给不同的应用去处理。这里的应用就是tap，pip\_to\_shower。

`egg:Paste#urlmap`实际是一个入口点，类似如下

```text
setup(
    name='Paste',
    # ...
    entry_points={
        'use': [
            'urlmap=myapp.mymodule:function',
        },
    )
```

### 2.2、app

```text
[app:tap]
paste.app_factory = tap:app_factory
in_arg = water
```

表示路径"/tap"的处理方法paste.app\_factory存在于tap.py文件的的app\_factory中，这是一个方法。

### 2.3、pipeline

```text
[pipeline:pip_to_shower]
pipeline = boiler shower
```

**pipeline** 主要起到组合的作用，将filter\(过滤器\)和app\(应用\)组合起来，形成一条管道

### 2.4、filter

```text
[filter:boiler]
paste.filter_app_factory = boiler:filter_app_factory
in_arg = water
```

filter类似app，只不过换成了paste.filter\_app\_factory，filter首先执行过滤功能，然后执行app。

## 3、参考文档

[PasteDeploy模块介绍](https://blog.csdn.net/meisanggou/article/details/88179254)

[nova创建虚拟机源码分析系列之三 PasteDeploy](https://www.cnblogs.com/goldsunshine/p/7756153.html)

#### [Paste.deploy](http://www.choudan.net/2013/07/28/OpenStack-paste-deploy%E4%BB%8B%E7%BB%8D.html) <a id="subnav-0"></a>



