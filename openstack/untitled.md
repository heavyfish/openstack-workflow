# OpenStack中的打包工具

## 1、概述

OpenStack的打包工具，用的是python的`setuptools`。

![](../.gitbook/assets/image%20%2853%29.png)

在打包时，opnestack中还引入了Pbr管理工具

> `pbr`是一个管理python setuptools 的工具库，pbr模块读入setup.cfg文件的信息，并且给setuptools 中的setup hook 函数填写默认参数，提供更加有意义的行为

1. `pbr`是`setuptools`的辅助工具，最初为openstack开发，基于d2to1。Pbr会读取和过滤setup.cfg中的内容，然后将解析后的数据提供给setup.py作为参数。
2. setup.cfg提供setup.py的默认参数，同时易于修改。Setup.py先解析setup.cfg文件，然后执行相关命令。包括以下功能：

```text
从git中获取Version，AUTHORS和ChangeLog信息
SphinxAutodoc。pbr会扫描project，找到所有模块，生成stubfiles
Requirements。读取requirements.txt文件，生成setup函数需要依赖包
long_description。从README.rst、README.txt或者READMEfile中生成long_description参数
```

## 2、参考文档

[python打包工具distutils、setuptools分析](https://www.cnblogs.com/goldsunshine/p/8872623.html)

