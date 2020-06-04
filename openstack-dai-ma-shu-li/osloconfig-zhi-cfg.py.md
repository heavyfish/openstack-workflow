---
description: P版
---

# oslo\_config之cfg.py

## 1、简介注释

配置项可以设置在命令行或配置文件中.

每个选项的架构都使用 class:\`Opt\` class or its sub-classes, 例如:

```text
from oslo_config import cfg
from oslo_config import types

PortType = types.Integer(1, 65535)

common_opts = [
    cfg.StrOpt('bind_host',
               default='0.0.0.0',
               help='IP address to listen on.'),
    cfg.Opt('bind_port',
            type=PortType,
            default=9292,
            help='Port number to listen on.')
]
```

### Option Types

通过 :class:`Opt`构造函数的`type`参数，options 可以具有任意类型。 `type`参数是一个可调用对象，该对象需要一个字符串，并返回该特定类型的值，如果无法转换该值，则引发 :class:`ValueError`。

```text
======================================  ======
Type                                    Option
======================================  ======
:class:`oslo_config.types.String`       :class:`oslo_config.cfg.StrOpt`
:class:`oslo_config.types.String`       :class:`oslo_config.cfg.SubCommandOpt`
:class:`oslo_config.types.Boolean`      :class:`oslo_config.cfg.BoolOpt`
:class:`oslo_config.types.Integer`      :class:`oslo_config.cfg.IntOpt`
:class:`oslo_config.types.Float`        :class:`oslo_config.cfg.FloatOpt`
:class:`oslo_config.types.Port`         :class:`oslo_config.cfg.PortOpt`
:class:`oslo_config.types.List`         :class:`oslo_config.cfg.ListOpt`
:class:`oslo_config.types.Dict`         :class:`oslo_config.cfg.DictOpt`
:class:`oslo_config.types.IPAddress`    :class:`oslo_config.cfg.IPOpt`
:class:`oslo_config.types.Hostname`     :class:`oslo_config.cfg.HostnameOpt`
:class:`oslo_config.types.HostAddress`  :class:`oslo_config.cfg.HostAddressOpt`
:class:`oslo_config.types.URI`          :class:`oslo_config.cfg.URIOpt`
======================================  ======
```

对于 :class:`oslo_config.cfg.MultiOpt` the `item_type` 参数定义了值的类型. 为了方便, :class:`oslo_config.cfg.MultiStrOpt` 等效于  :class:`~oslo_config.cfg.MultiOpt` 且`item_type` 参数设置为 :class:`oslo_config.types.MultiString`.

以下示例使用便捷类定义选项:

```text
enabled_apis_opt = cfg.ListOpt('enabled_apis',
                               default=['ec2', 'osapi_compute'],
                               help='List of APIs to enable by default.')

DEFAULT_EXTENSIONS = [
    'nova.api.openstack.compute.contrib.standard_extensions'
]
osapi_compute_extension_opt = cfg.MultiStrOpt('osapi_compute_extension',
                                              default=DEFAULT_EXTENSIONS)
```

### Registering Options

Option schemas在该选项被引用前，已在配置管理器中注册

```text
class ExtensionManager(object):

    enabled_apis_opt = cfg.ListOpt(...)

    def __init__(self, conf):
        self.conf = conf
        self.conf.register_opt(enabled_apis_opt)
        ...

    def _load_extensions(self):
        for ext_factory in self.conf.osapi_compute_extension:
            ....
```

对于在使用该选项的模块或类中定义的每个 选项模式（option schema），常见的用法是

```text
opts = ...

def add_common_opts(conf):
    conf.register_opts(opts)

def get_bind_host(conf):
    return conf.bind_host

def get_bind_port(conf):
    return conf.bind_port
```

可以选择通过命令行提供一个选项。 在解析命令行之前，必须在配置管理器中注册这些选项（为了--help和CLI arg可被验证）

```text
cli_opts = [
    cfg.BoolOpt('verbose',
                short='v',
                default=False,
                help='Print more verbose output.'),
    cfg.BoolOpt('debug',
                short='d',
                default=False,
                help='Print debugging output.'),
]

def add_common_opts(conf):
    conf.register_cli_opts(cli_opts)
```

### Loading Config Files

配置管理器默认情况下定义了两个CLI选项，--config-file和--config-dir

```text
class ConfigOpts(object):

    def __call__(self, ...):

        opts = [
            MultiStrOpt('config-file',
                    ...),
            StrOpt('config-dir',
                   ...),
        ]

        self.register_cli_opts(opts)
```

使用`oslo_config.iniparser`可从任何提供的配置文件中解析选项值。 如果未指定配置文件，则使用默认集，例如glance-api.conf和glance-common.conf

```text
glance-api.conf:
  [DEFAULT]
  bind_port = 9292

glance-common.conf:
  [DEFAULT]
  bind_host = 0.0.0.0
```

配置文件中的行不应以空格开头。 配置文件还支持注释，注释必须以“＃”或“;”开头。 配置文件中的选项值和命令行中的选项值将按顺序进行解析。 同一选项（包括`deprecated option name`和`current option name`）可以在配置文件或命令行中多次出现。 后面的值始终会覆盖前面的值

同一配置目录中配置文件的顺序由其文件名的字母排序顺序定义。

CLI args和配置文件的解析是通过调用配置管理器启动的，例如

```text
conf = cfg.ConfigOpts()
conf.register_opt(cfg.BoolOpt('verbose', ...))
conf(sys.argv[1:])
if conf.verbose:
    ...
```

### Option Groups

可以将选项注册为属于组:

```text
rabbit_group = cfg.OptGroup(name='rabbit',
                            title='RabbitMQ options')

rabbit_host_opt = cfg.StrOpt('host',
                             default='localhost',
                             help='IP/hostname to listen on.'),
rabbit_port_opt = cfg.PortOpt('port',
                              default=5672,
                              help='Port number to listen on.')

def register_rabbit_opts(conf):
    conf.register_group(rabbit_group)
    # options can be registered under a group in either of these ways:
    conf.register_opt(rabbit_host_opt, group=rabbit_group)
    conf.register_opt(rabbit_port_opt, group='rabbit')
```

如果除组名外不需要任何组属性，则无需显式注册该组，例如：:

```text
def register_rabbit_opts(conf):
    # The group will automatically be created, equivalent calling::
    #   conf.register_group(OptGroup(name='rabbit'))
    conf.register_opt(rabbit_port_opt, group='rabbit')
```

如果未指定任何组，则选项属于配置文件的“ DEFAULT”部分:

```text
glance-api.conf:
  [DEFAULT]
  bind_port = 9292
  ...

  [rabbit]
  host = localhost
  port = 5672
  use_ssl = False
  userid = guest
  password = guest
  virtual_host = /
```

组中的命令行选项会自动以组名作为前缀:

```text
--rabbit-host localhost --rabbit-port 9999
```

### Dynamic Groups

可以通过应用程序代码动态注册组。 这给样本生成器（sample generator），发现机制和验证工具带来了挑战，因为它们事先不知道所有组的名称。 构造函数的`dynamic_group_owner`参数指定在另一个组中注册的选项的全名，该选项控制动态组的重复实例。 此选项通常是MultiStrOpt。

例如，Cinder支持多个存储后端设备和服务。 要配置Cinder与多个后端通信，将`enabled_backends`选项设置为后端名称列表。 每个后端组均包含用于与该设备或服务进行通信的选项。

### Driver Groups

组可以具有动态的选项集，通常基于具有独特要求的驱动程序。 这在运行时有效，因为代码在使用选项之前先对其进行注册，但是由于样本生成器，发现机制和验证工具无法事先知道组的正确选项，因此给样本生成器，发现机制和验证工具带来了挑战。

要解决此问题，可以使用`driver_option`参数来命名组的驱动程序选项。 每个驱动程序选项都应定义自己的发现入口点名称空间，以返回该驱动程序的选项集，并使用前缀`“ oslo.config.opts”`命名，后跟驱动程序选项名称。

在上述Cinder示例中，`volume_backend_name`选项是该组的静态定义的一部分，因此应该将`driver_option`设置为“ `volume_backend_name`”。 插件应使用与`“ oslo.config.opts”`注册的主插件相同的名称在`“ oslo.config.opts.volume_backend_name”`下注册。 驻留在Cinder代码库中的驱动程序已注册名为`“ cinder”`的入口点。

### Accessing Option Values In Your Code

默认组中的选项值在配置管理器中被引用为attributes/properties 。 组也是配置管理器上的属性，具有与该组关联的每个选项的属性：

```text
server.start(app, conf.bind_port, conf.bind_host, conf)

self.connection = kombu.connection.BrokerConnection(
    hostname=conf.rabbit.host,
    port=conf.rabbit.port,
    ...)
```

### Option Value Interpolation

选项值可以使用PEP 292字符串替换引用其他值：

```text
opts = [
     cfg.StrOpt('state_path',
               default=os.path.join(os.path.dirname(__file__), '../'),
               help='Top-level directory for maintaining nova state.'),
    cfg.StrOpt('sqlite_db',
               default='nova.sqlite',
               help='File name for SQLite.'),
    cfg.StrOpt('sql_connection',
               default='sqlite:///$state_path/$sqlite_db',
               help='Connection string for SQL database.'),
]
```

> Interpolation can be avoided by using `$$`
>
> 您可以使用 `.` 将选项与其他群组分隔开，例如  $ {mygroup.myoption}

### Special Handling Instructions

可以根据需要声明选项，以便在用户不提供选项值的情况下引发错误：

```text
opts = [
    cfg.StrOpt('service_name', required=True),
    cfg.StrOpt('image_id', required=True),
    ...
]
```

选项可以声明为`secr`，这样它们的值就不会泄漏到日志文件中:

```text
 opts = [
    cfg.StrOpt('s3_store_access_key', secret=True),
    cfg.StrOpt('s3_store_secret_key', secret=True),
    ...
 ]
```

### Dictionary Options

如果您需要最终用户指定键/值对的字典，则可以使用DictOpt:

```text
opts = [
    cfg.DictOpt('foo',
                default={})
]
```

然后，最终用户可以在其配置文件中指定选项foo（ini格式），如下所示

```text
[DEFAULT]
foo = k1:v1,k2:v2
```

### Global ConfigOpts

该模块还包含ConfigOpts类的全局实例，以支持OpenStack中的常见使用：

```text
from oslo_config import cfg

opts = [
    cfg.StrOpt('bind_host', default='0.0.0.0'),
    cfg.PortOpt('bind_port', default=9292),
]

CONF = cfg.CONF
CONF.register_opts(opts)

def start(server, app):
    server.start(app, CONF.bind_port, CONF.bind_host)
```

### Positional Command Line Arguments

位置命令行参数通过“positional” Opt构造函数参数支持：

```text
>>> conf = cfg.ConfigOpts()
>>> conf.register_cli_opt(cfg.MultiStrOpt('bar', positional=True))
True
>>> conf(['a', 'b'])
>>> conf.bar
['a', 'b']
```

### Sub-Parsers

也可以使用SubCommandOpt类使用 argparse“ sub-parsers”来解析其他命令行参数：

```text
>>> def add_parsers(subparsers):
...     list_action = subparsers.add_parser('list')
...     list_action.add_argument('id')
...
>>> conf = cfg.ConfigOpts()
>>> conf.register_cli_opt(cfg.SubCommandOpt('action', handler=add_parsers))
True
>>> conf(args=['list', '10'])
>>> conf.action.name, conf.action.id
('list', '10')

```

### Advanced Option

如果需要在示例文件中将选项标记为高级，则使用，但大多数用户通常不使用该选项，并且可能对稳定性或性能产生重大影响

```text
from oslo_config import cfg

opts = [
    cfg.StrOpt('option1', default='default_value',
                advanced=True, help='This is help '
                'text.'),
    cfg.PortOpt('option2', default='default_value',
                 help='This is help text.'),
]

CONF = cfg.CONF
CONF.register_opts(opts)
```

这将导致该选项被推到名称空间的底部，并在示例文件中标记为Advanced，并带有可能有影响的符号：

```text
[DEFAULT]
...
# This is help text. (string value)
# option2 = default_value
...
<pushed to bottom of section>
...
# This is help text. (string value)
# Advanced Option: intended for advanced users and not used
# by the majority of users, and might have a significant
# effect on stability and/or performance.
# option1 = default_value
```

### Option Deprecation

如果要重命名某些选项，将它们移至另一个组或完全删除，则可以使用`deprecated_name`，`deprecated_group`和`deprecated_for_removal`参数将其声明更改为 :class:`Opt` 构造函数

```text
from oslo_config import cfg

conf = cfg.ConfigOpts()

opt_1 = cfg.StrOpt('opt_1', default='foo', deprecated_name='opt1')
opt_2 = cfg.StrOpt('opt_2', default='spam', deprecated_group='DEFAULT')
opt_3 = cfg.BoolOpt('opt_3', default=False, deprecated_for_removal=True)

conf.register_opt(opt_1, group='group_1')
conf.register_opt(opt_2, group='group_2')
conf.register_opt(opt_3)

conf(['--config-file', 'config.conf'])

assert conf.group_1.opt_1 == 'bar'
assert conf.group_2.opt_2 == 'eggs'
assert conf.opt_3
```

假设文件config.conf具有以下内容

```text
[group_1]
opt1 = bar

[DEFAULT]
opt_2 = eggs
opt_3 = True
```

该脚本将成功执行，但是将记录有关给定不推荐使用的选项的三个警告

还有`deprecated_reason`和`deprecated_since`参数，用于指定有关弃用的其他一些信息。

所有提到的参数可以任意组合混合在一起。

### 2、概念解析

### 2.1、名词解释

**配置文件**：

用来配置OpenStack各个服务的ini风格的配置文件，通常以`.conf`结尾；

**配置项\(options\)：**

配置文件或命令行中给出的配置信息的左值，如：`enabled_apis = ec2, osapi_keystone,osapi_compute`中的“`enabled_apis`”；

**配置项的值：**

配置文件或命令行中给出的配置信息的右值，如：`enabled_apis =ec2, osapi_keystone, osapi_compute`中的“`ec2,osapi_keystone, osapi_compute`”；

**配置组\(option groups\)：**

一组配置项，在配置文件中通过`[...]`来表示，如my.conf文件中的`[rabbit]`字段表示接下来开始一个名为rabbit的配置组； 默认配置组和添加的配置组

**配置项模式\(option schemas\)：**

在解析配置文件、获取配置项的值之前，其他模块声明自己需要的配置项。配置文件通常是针对一个完整的服务的，因此其他模块中可能用不到配置文件中的所有配置项，这样就必须告诉系统自己依赖于哪些配置项，这个过程就是设置配置项的模式。包括声明配置项在配置文件的名称、设置配置项的默认值（一旦配置文件中没有该配置项而其他模块又依赖于该配置项，就使用这里声明的默认值）等等；

**引用\(reference\)：**

其他模块解析配置文件，获取配置项的值后，就可以在下面的实现中使用这些具体的配置值了；

**注册\(register\)：**

其他模块在引用配置项的值之前，必须注册自己将要引用的那些配置项的模式。也就是说，配置文件中的配置项其他模块不一定都为其声明模式，声明了模式的配置项也不一定为其进行注册，当然如果不注册，即使声明了模式，也无法引用。

### 2.2、解析步骤

下面先给一个high-level的过程说明一下如何使用这个库，OpenStack中配置文件的解析主要有以下几个步骤：

step1. 正确配置服务的主配置文件（\*.conf文件），本步骤在各个服务（如：keystone）中完成。

step2. 在要使用到配置信息的模块中声明将用到的那些配置项的模式，包括配置项的名称、数据类型、默认值和说明等；

```python
enabled_apis_opt = cfg.ListOpt('enabled_apis',
           default=['ec2', 'osapi_compute'],
           help='List of APIs to enable by default.')
```

step3. 创建一个对象，创建该对象的类充当配置管理器，这个对象作为容器以后将存储配置项的值。`cfg.CONF` 是全局配置项管理类。

```python
CONF = cfg.CONF
```

step4. 调用step3创建的对象中相应的注册方法（如：register\_opt\(\)），注册step2中声明的配置项模式。这个过程不会解析配置文件，只是为step3中创建的对象开辟相应的字段。

```python
CONF.register_opt(enabled_apis_opt)
```

step5. 直接调用step3中创建的对象，传入配置文件路径等信息。此时将会解析配置文件，如果未指定配置文件则全部使用step2模式中的默认值。解析过程会提取step4中注册了的配置项的值，然后这些配置项就作为step3创建的对象的属性可以被直接引用。

```python
if __name__ =="__main__":
    # 调用容器对象，传入要解析的文件（可以多个）
　　CONF(default_config_files=['my.conf'])
```

```python
#cfg.CONF是oslo.config中定义的全局对象实例，  是一个配置项的管理类 
from oslo.config import cfg

# 声明配置项模式
# 单个配置项模式  配置项 配置项的值
enabled_apis_opt = cfg.ListOpt('enabled_apis',
           default=['ec2', 'osapi_compute'],
           help='List of APIs to enable by default.')

# 多个配置项组成一个模式
common_opts = [
        cfg.StrOpt('bind_host',
        default='0.0.0.0',
        help='IP address to listen on.'),
                
        cfg.IntOpt('bind_port',
        default=8080,
        help='Port number to listen on.')
    ]

# 自定义配置组
rabbit_group = cfg.OptGroup(
    name='rabbit',
    title='RabbitMQ options'
)

# 配置组中的模式，通常以配置组的名称为前缀（非必须）  单个配置项模式
rabbit_ssl_opt = cfg.BoolOpt('use_ssl',
          default=False,
          help='use ssl for connection')

# 配置组中的多配置项模式
rabbit_Opts = [
    cfg.StrOpt('host',
         default='localhost',
         help='IP/hostname to listen on.'),
    cfg.IntOpt('port',
         default=8000,
         help='Port number to listen on.')
]
 
# 创建对象CONF，用来充当容器，管理配置项的类实例
CONF = cfg.CONF

# 注册单个配置项模式
CONF.register_opt(enabled_apis_opt)
 
# 注册含有多个配置项的模式
CONF.register_opts(common_opts)
 
# 配置组必须在其配置项被注册前  注册！   注册自定义配置组
CONF.register_group(rabbit_group)
 
# 注册配置组中含有多个配置项的模式，必须指明配置组
CONF.register_opts(rabbit_Opts, rabbit_group)
 
# 注册配置组中的单配置项模式，指明配置组
CONF.register_opt(rabbit_ssl_opt, rabbit_group)
 
# 接下来打印使用配置项的值
if __name__ =="__main__":
    # 调用容器对象，传入要解析的文件（可以多个）
　　CONF(default_config_files=['my.conf'])
     
    for i in CONF.enabled_apis:
        print "DEFAULT.enabled_apis: " + i
     
    print "DEFAULT.bind_host: " + CONF.bind_host
    print "DEFAULT.bind_port: " + str(CONF.bind_port)
    print "rabbit.use_ssl: "+ str(CONF.rabbit.use_ssl)
    print "rabbit.host: " + CONF.rabbit.host
    print "rabbit.port: " + str(CONF.rabbit.port)
```

## 3、参考文档

简介注释源于  cfg.py 的 注释

[OpenStack oslo\_config 库](https://blog.csdn.net/li_101357/article/details/70159045)

