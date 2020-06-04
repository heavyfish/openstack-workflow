---
description: 对Apache的简单介绍
---

# Apache

## 1、基本介绍

Apache是一个web服务器。apache2以上版本改称为httpd。所以现在Linux中安装是

```text
yum install httpd
```

httpd也是Apache的主程序，它被设计为一个独立运行的后台进程，它会建立一个处理请求的子进程或线程的池

而通过yum安装的httpd服务的主配置路径如下：

```text
httpd-2.2：/etc/httpd/conf/httpd.conf
httpd-2.4：/etc/httpd/conf/httpd.conf
```

通常在主配置文件中会使用`Include conf.d/*.conf`、`Include conf.modules.d/*.conf`（httpd-2.4）类似的语句去调用对应的目录下的配置文件，其路径格式为相对路径，根目录由主配置文件中的Serverroot决定。

主配置文件的格式大体分为三部分：

> Section 1: Global Environment  
>  Section 2: 'Main' server configuration  
>  Section 3: Virtual Hosts

## 2、目录结构

![](../.gitbook/assets/image%20%2814%29.png)

### 2.1、conf

![](../.gitbook/assets/image%20%2830%29.png)

httpd.conf：主配置文件

magic：供模块 mod\_mime\_magic 使用。当 mod\_mime无法解析时，检查文件开始的几个字节，来判定文件的[MIME类型](http://www.jinbuguo.com/apache/manual/glossary.html#mime-type)。

### 2.2、conf.d

主配置文件 通过 Include 命令，调用其下的配置文件。其目录包含Apache HTTP Server的配置文件。 该目录中所有扩展名为“ .conf”的文件都将是 作为httpd配置文件处理。

### 2.3、conf.modules.d

。其中包含 加载模块所需的配置文件，需要加载相关模块时，可修改其中的文件，然后重启httpd服务

### 2.4、link文件

logs：日志目录

moduels：module目录

run：运行时目录

## 3、配置文件解析

### 3.1、httpd.conf

```bash
#http安装时的 base directory
ServerRoot "/etc/httpd"

#服务器侦听的IP地址和端口
Listen 12.34.56.78:80

#导入其他配置文件
Include conf.modules.d/*.conf

#运行http的user/group httpd
User apache
Group apache

#服务器在发送给客户端的错误消息中包含的电子邮件地址
ServerAdmin root@localhost

#包含一组指令，仅适用于 指定文件系统目录，子目录及其内容
<Directory />
    AllowOverride none
    Require all denied
</Directory>

#包含一组指令，处理前，需先判断特定模块是否存在
<IfModule dir_module>
    DirectoryIndex index.html
</IfModule>

#<Files>指令通过文件名来限制所附指令的范围
<Files ".ht*">
    Require all denied
</Files>

#记录错误日志的位置。 此处路径是 相对于 Base directory的相对路径
ErrorLog "logs/error_log"

#日志等级
LogLevel warn

#respose content-type 为text/plain或text/html时
#要添加的默认charset参数
AddDefaultCharset UTF-8

#使用内核sendfile支持将文件传送到客户端
EnableSendfile on

#include包含的文件不存在时，将报错。
#使用IncludeOptional指令进行加载，表示存在则加载，不存在就算了
IncludeOptional conf.d/*.conf
```

### 3.2、autoindex.conf

其中的指令控制服务器生成的目录列表

```bash
#控制服务器生成的目录的样式
IndexOptions FancyIndexing HTMLTable VersionSort

#映射 url 到本地文件系统
Alias /icons/ "/usr/share/httpd/icons/"

#在MIME content-encoding 选择的文件旁边显示的图标
AddIconByEncoding (CMP,/icons/compressed.gif) x-compress x-gzip

#在MIME conten-type选择的文件旁边显示的图标
AddIconByType (TXT,/icons/text.gif) text/*

#为按name选择的文件显示的图标
AddIcon /icons/binary.gif .bin .exe

#未配置任何特定图标时显示的文件图标
DefaultIcon /icons/unknown.gif

#将在索引列表末尾插入的文件名
ReadmeName README.htm

#将在索引列表顶部插入的文件的名称
HeaderName HEADER.html

#列出索引目录时将隐藏的文件
IndexIgnore .??* *~ *# HEADER* README* RCS CVS *,v *,t
```

### 3.3、ssl.conf

与配置ssl有关。是[mod\_ssl](http://httpd.apache.org/docs/2.4/mod/mod_ssl.html)的配置文件

### 3.4、usedir.conf

[mod\_userdir](http://httpd.apache.org/docs/2.4/mod/mod_userdir.html)的配置文件。mod\_usedir允许使用http://example.com/~user/语法访问特定于用户的目录

### 3.5、welcome.conf

如果存在，此配置文件将启用默认的“欢迎”页面 是 root URL 的默认索引页

## 4、OpenStack相关

以P版DevStack部署环境为例

### 4.1、keystone-wsgi-admin.conf

```bash
# 设置一个内部环境变量，
# 该变量随后可用于Apache HTTP Server模块，并传递给CGI脚本和SSI页面。
SetEnv proxy-sendcl 1

#将远程服务器映射到本地服务器的空间
#本地服务器不充当常规意义上的代理，而是远程服务器的镜像
#本地服务器称为反向代理或gataway
# /indentity_admin 是本地虚拟路径的名称
# unix:... 是远程服务器的部分url
#此处会将所有对 /identity_admin的请求， 代理到后侧url
ProxyPass "/identity_admin" "unix:/var/run/uwsgi/keystone-wsgi-admin.socket|uwsgi://uwsgi-uds-keystone-wsgi-admin/" retry=0
```

### 4.2、horizon.conf

```bash
#包含仅适用于特定主机名或IP地址的指令
<VirtualHost *:80>
    #将URL映射到文件系统位置，并将目标指定为WSGI脚本
    WSGIScriptAlias /dashboard /opt/stack/horizon/openstack_dashboard/wsgi/django.wsgi
    
    #用于指定应创建不同的守护进程，可以将WSGI application的运行委托给该守护进程
    WSGIDaemonProcess horizon user=stack group=stack processes=3 threads=10 home=/opt/stack/horizon display-name=%{GROUP}
    
    #设置WSGI application所属的 application group
    WSGIApplicationGroup %{GLOBAL}

    #设置变量  
    SetEnv APACHE_RUN_USER stack
    SetEnv APACHE_RUN_GROUP stack
    
    #设置将WSGI application分配给哪个进程组
    WSGIProcessGroup horizon

    #构成从网络可见的主文档树目录
    DocumentRoot /opt/stack/horizon/.blackhole/
    
    #将URL映射到文件系统位置
    Alias /dashboard/media /opt/stack/horizon/openstack_dashboard/static
    Alias /dashboard/static /opt/stack/horizon/static

    #根据当前URL的正则表达式匹配发送外部重定向
    #这里是凡是访问/，就跳转到/dashboard/
    RedirectMatch "^/$" "/dashboard/"

    #包含一组指令，仅适用于 指定文件系统目录，子目录及其内容
    <Directory />
        
        #配置特定目录中可用的功能
        #服务器将遵循此目录中的符号链接。这是默认设置。
        Options FollowSymLinks
        
        #禁用.htaccess文件中允许的指令类型
        AllowOverride None
    </Directory>

    
    <Directory /opt/stack/horizon/>
        #Indexes：如果请求映射到目录的URL，
        #并且该目录中没有DirectoryIndex（例如index.html）
        #则mod_autoindex将返回该目录的格式列表。
        #MultiViews：内容协商，被允许使用mod_negotiation
        Options Indexes FollowSymLinks MultiViews
        AllowOverride None
        # Apache 2.4 uses mod_authz_host for access control now (instead of
        #  "Allow")
        <IfVersion < 2.4>
            
            #控制默认访问状态以及评估“allow”和“deny”的顺序。
            #首先，评估所有Allow指令；至少一个必须匹配，否则请求被拒绝。
            #接下来，将评估所有“deny”指令。如果有匹配项，则请求被拒绝。
            #最后，默认情况下会拒绝所有与Allow或Deny指令不匹配的请求。
            Order allow,deny
            Allow from all
        </IfVersion>
        <IfVersion >= 2.4>
            #无条件允许访问。
            Require all granted
        </IfVersion>
    </Directory>
    <IfVersion >= 2.4>
      ErrorLogFormat "%{cu}t %M"
    </IfVersion>
    ErrorLog /var/log/httpd/horizon_error.log
    LogLevel warn
    CustomLog /var/log/httpd/horizon_access.log combined
</VirtualHost>

#配置要用于守护程序套接字的目录
WSGISocketPrefix /var/run/httpd
```

> [mod\_wsgi](https://modwsgi.readthedocs.io/en/develop/configuration-directives/WSGISocketPrefix.html)\(WSGI开头的配置源于此模块\)

## 参考文档

[apache官网](http://httpd.apache.org/docs/2.4/)

[httpd的配置文件常见设置](https://www.cnblogs.com/struggle-1216/p/11944064.html)（可看其中关于modules的描述）

[http.conf配置说明](https://www.cnblogs.com/f-ck-need-u/p/7636836.html)（对部分配置有讲解说明）

[httpd服务的配置及应用](https://www.jianshu.com/p/687b915766b6)（有部分使用案例）

