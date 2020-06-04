---
description: 对于《Django2.0入门与实践》中搭建过程的记录
---

# Django2.0搭建

## 1、环境准备

### 1.1、yum 源准备

```text
curl -o CentOS-Base.repo http://mirrors.163.com/.help/CentOS7-Base-163.repo

yum install epel-release

yum install python2-pip-8.1.2-10.el7.noarch
```

### 1.2、修改pip源

1）到`~/.config/pip`下新建`pip.conf`文件，并写入以下内容

```text
[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple

[install]
# trusted-host用来避免麻烦，否则某些源会提示不受信任
trusted-host=pypi.tuna.tsinghua.edu.cn
```

国内几个常用的源

```text
阿里云 http://mirrors.aliyun.com/pypi/simple/
中国科技大学 https://pypi.mirrors.ustc.edu.cn/simple/
豆瓣(douban) http://pypi.douban.com/simple/
清华大学 https://pypi.tuna.tsinghua.edu.cn/simple/
中国科学技术大学 http://pypi.mirrors.ustc.edu.cn/simple/
```

2）通过命令行也可进行pip源的修改

```text
# 临时使用
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple some-package
```

```text
# 设为默认
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple pip -U
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
```

### 1.3、安装python3.7

1）登录[https://www.python.org/downloads/source/](https://www.python.org/downloads/source/)，找到对应版本如图：

![](../.gitbook/assets/image%20%2813%29.png)

```text
wget https://www.python.org/ftp/python/3.7.6/Python-3.7.6.tgz
```

2）上传到了`/root/tools`目录下，解压`tar -zxvf Python-3.7.6.tgz`

3）安装依赖

```text
yum -y install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gcc make

# python3.7开始需要安装 libffi-devel
yum install libffi-devel
```

4）进入解压后的目录，编译安装

```text
./configure
make
make install
```

### 1.4、创建虚拟环境

```text
pip3 install virtualenv
```

```text
# 修改 ~/.bashrc，加入如下代码，重登终端
export VIRTUALENVWRAPPER_PYTHON=/usr/bin/python3
export VIRTUALENVWRAPPER_VIRTUALENV=/usr/local/bin/virtualenv
```

```text
#创建虚拟环境
virtualenv <venv_name> --python=python3
```

### 1.5、安装django2.0

1）进入虚拟环境

```text
pip install django==2.0
```

### 1.6、安装mariadb

```text
yum install -y mariadb mariadb-server

#初始化数据库
mysql_secure_installation
```

### 1.7、修改数据库配置

```text
[mysqld]

character-set-server=utf8mb4

[mysql]

default-character-set=utf8mb4
```

## 2、所遇错误

### 2.1、无 mysqlclient 报错

```text
django.core.exceptions.ImproperlyConfigured: Error loading MySQLdb module.
Did you install mysqlclient?
```

解决方法：用pymysql代替

```text
# pip install pymysql

#在mysite下的 __init__.py 添加
import pymysql
pymysql.install_as_MySQLdb()
```

### 2.2、django无法修改绑定IP

```text
python manage.py runserver 172.16.29.63:8000
```

![](../.gitbook/assets/image%20%284%29.png)

解决方法：

```text
修改mystie下的settings.py。  ['*']：表示允许多有主机访问django

ALLOWED_HOSTS = ['*']
```

### 2.3、polls/urls.py

django报错：'set' object is not reversible

```text
# 注意 应该是 {} 
urlpatterns = {
    path('', views.index, name='index'),
}
```





