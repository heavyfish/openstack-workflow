---
description: 测试环境为P版的DevStack
---

# Nova代码梳理-P版

## 1、HTTP

### 1.1、请求转发

P版中有httpd和uwsgi。在keystone和nova访问相应组件时，经过了一层httpd的代理。此处以keystone为例。

查看keystone服务器的启动命令，是用的uwsgi

![](../.gitbook/assets/image%20%2857%29.png)

当访问keystone的REST API接口时，请求被代理到了uwsgi处理

![](../.gitbook/assets/image%20%2858%29.png)

当在用pdb调试nova-api时时，需要对这里进行修改，才能卡住api层的代码

![](../.gitbook/assets/image%20%2816%29.png)

## 2、nova-api

nova-api的启动文件在 `nova/api/cmd/api.py` 中，下面进行分析

```text
import sys

from oslo_log import log as logging
from oslo_reports import guru_meditation_report as gmr

import nova.conf
from nova import config
from nova import exception
from nova import objects
from nova import service
from nova import utils
from nova import version

CONF = nova.conf.CONF


def main():
    config.parse_args(sys.argv)
    logging.setup(CONF, "nova")
    utils.monkey_patch()
    objects.register_all()
    if 'osapi_compute' in CONF.enabled_apis:
        # NOTE(mriedem): This is needed for caching the nova-compute service
        # version.
        objects.Service.enable_min_version_cache()
    log = logging.getLogger(__name__)

    gmr.TextGuruMeditation.setup_autorun(version)

    launcher = service.process_launcher()
    started = 0
    for api in CONF.enabled_apis:
        should_use_ssl = api in CONF.enabled_ssl_apis
        try:
            server = service.WSGIService(api, use_ssl=should_use_ssl)
            launcher.launch_service(server, workers=server.workers or 1)
            started += 1
        except exception.PasteAppNotFound as ex:
            log.warning("%s. ``enabled_apis`` includes bad values. "
                        "Fix to remove this warning.", ex)

    if started == 0:
        log.error('No APIs were started. '
                  'Check the enabled_apis config option.')
        sys.exit(1)

    launcher.wait()
```

### 2.1、导入模块

```text
import sys

from oslo_log import log as logging
from oslo_reports import guru_meditation_report as gmr

import nova.conf
from nova import config
from nova import exception
from nova import objects
from nova import service
from nova import utils
from nova import version
```

* sys：sys模块提供了一系列有关Python运行环境的变量和函数
* oslo\_log：oslo.log（logging）配置库为所有openstack项目提供了标准化的配置。 它还提供了自定义formatters，handlers以及对特定于上下文的日志记录（如资源ID的支持）的支持
* oslo\_reports：在OpenStack的（生产）部署中出现问题时，收集调试数据是分类和最终解决问题的关键的第一步。 像Nova这样的项目已广泛使用了日志记录功能，该功能可生成大量数据。 但是，这不能使管理员获得有关系统当前活动状态的准确视图。 例如，正在运行哪些线程，有效的配置参数等等。项目oslo.reports托管了一个通用的错误报告生成框架，称为“guru meditation report”（参见[http://en.wikipedia.org/wiki/Guru\_Meditation）](http://en.wikipedia.org/wiki/Guru_Meditation），用于解决上述问题)，用于解决上述问题

### 2.2、CONF

```text
CONF = nova.conf.CONF
```

进入到 `nova/conf/__init__.py`。有关代码是

```python
from oslo_config import cfg
from nova.conf import api
...

CONF = cfg.CONF
api.register_opts(CONF)

...
remote_debug.register_cli_opts(CONF)
```

> oslo.config库用于**解析命令行和配置文件中的配置选项**，把配置项直接融入代码内

由此跳转到`/usr/lib/python2.7/site-packages/oslo_config/cfg.py`。

```python
CONF = ConfigOpts()
```

```python
class ConfigOpts(collections.Mapping):

    """Config选项，可以在命令行或配置文件中设置.

    ConfigOpts 是一个 configuration option manager 带有API 用于注册
    option schemas, grouping options, parsing option values
    和 检索 options values.

    It has built-in support 对于 :oslo.config:option:`config_file` and
    :oslo.config:option:`config_dir` options.

    """
    disallow_names = ('project', 'prog', 'version',
                      'usage', 'default_config_files', 'default_config_dirs')

    def __init__(self):
        """Construct a ConfigOpts object."""
        self._opts = {}  # dict of dicts of (opt:, override:, default:)
        self._groups = {}
        self._deprecated_opts = {}

        self._args = None

        self._oparser = None
        self._namespace = None
        self._mutable_ns = None
        self._mutate_hooks = set([])
        self.__cache = {}
        self._config_opts = []
        self._cli_opts = collections.deque()
        self._validate_default_values = False
    ...
```

从上可看到，CONF实际上是个对象。然后是

```python
# nova/conf/__init__.py

from nova.conf import api
api.register_opts(CONF)
```

跳转到

```python
# nova/conf/api.py
from oslo_config import cfg

api_group = cfg.OptGroup('api',
    title='API options',
    help="""
Options under this group are used to define Nova API.
""")

auth_opts = [
    cfg.StrOpt("auth_strategy",
        default="keystone",
        choices=("keystone", "noauth2"),
        deprecated_group="DEFAULT",
        help="""
This determines the strategy to use for authentication: keystone or noauth2.
'noauth2' is designed for testing only, as it does no actual credential
checking. 'noauth2' provides administrative credentials only if 'admin' is
specified as the username.
"""),
    cfg.BoolOpt("use_forwarded_for",
        default=False,
        deprecated_group="DEFAULT",
        help="""
When True, the 'X-Forwarded-For' header is treated as the canonical remote
address. When False (the default), the 'remote_address' header is used.

You should only enable this if you have an HTML sanitizing proxy.
"""),
]

...

API_OPTS = (auth_opts +
            metadata_opts +
            file_opts +
            osapi_opts +
            allow_instance_snapshots_opts +
            osapi_hide_opts +
            fping_path_opts +
            os_network_opts +
            enable_inst_pw_opts)


def register_opts(conf):

    conf.register_group(api_group)
    conf.register_opts(API_OPTS, group=api_group)
    conf.register_opts(deprecated_opts)

...
```

形如`cfg.OptGroup()`，`cfg.StrOpt()`，`cfg.BoolOpt()`，都是cfg.py中的类型，它们都会返回一个对象，对象中，将创传进去的参数作为属性保存了下来，并会初始化其他属性。

```python
#OptGroup()相对简单，可看到它会初始化属性字段，并保留传进来的参数

class OptGroup(object):

    def __init__(self, name, title=None, help=None,
                 dynamic_group_owner='',
                 driver_option=''):
        """Constructs an OptGroup object."""
         
        self.name = name
        self.title = "%s options" % name if title is None else title
        self.help = help
        self.dynamic_group_owner = dynamic_group_owner
        self.driver_option = driver_option

        self._opts = {}  # dict of dicts of (opt:, override:, default:)
        self._argparse_group = None
        self._driver_opts = {}  # populated by the config generator
```

```python
API_OPTS = (auth_opts +
            metadata_opts +
            file_opts +
            osapi_opts +
            allow_instance_snapshots_opts +
            osapi_hide_opts +
            fping_path_opts +
            os_network_opts +
            enable_inst_pw_opts)
```

API\_OPTS的值则是list中包裹着object：\[&lt;object&gt;，&lt;object&gt;\]

![](../.gitbook/assets/image%20%2834%29.png)

```python
def register_opts(conf):

    conf.register_group(api_group)
    conf.register_opts(API_OPTS, group=api_group)
    conf.register_opts(deprecated_opts)
```

开始注册，首先是填充了`__groups`：

```python
    # conf.register_group(api_group)
    # 初始化时，self._groups = {}
    def register_group(self, group):
        """Register an option group.

        An option group must be registered before options can be registered
        with the group.

        :param group: an OptGroup object
        """
         
        if group.name in self._groups:
            return
        
        #copy.copy 浅拷贝
        self._groups[group.name] = copy.copy(group)
```

开始注册opts，注意每个opts 对象都会被单独注册

```python
   # conf.register_opts(API_OPTS, group=api_group)
    @__clear_cache
    def register_opts(self, opts, group=None):
        """Register multiple option schemas at once."""
         
        for opt in opts:
            self.register_opt(opt, group, clear_cache=False)
```

```python
    @__clear_cache
    def register_opt(self, opt, group=None, cli=False):
        """Register an option schema.

        注册 option schema 使先前或随后
        从命令行或配置文件解析的任何选项值都可用作该对象的属性.

        :param opt: an instance of an Opt sub-class
        :param group: an optional OptGroup object or group name
        :param cli: whether this is a CLI option
        :return: 如果已注册，则为False，否则为True
        :raises: DuplicateOptError
        """
         
        if group is not None:
            group = self._get_group(group, autocreate=True)
            if cli:
                self._add_cli_opt(opt, group)
            self._track_deprecated_opts(opt, group=group)
            return group._register_opt(opt, cli)

        # NOTE(gcb) We can't use some names which are same with attributes of
        # Opts in default group. They includes project, prog, version, usage,
        # default_config_files and default_config_dirs.
        if group is None:
            if opt.name in self.disallow_names:
                raise ValueError('Name %s was reserved for oslo.config.'
                                 % opt.name)

        if cli:
            self._add_cli_opt(opt, None)

        if _is_opt_registered(self._opts, opt):
            return False

        self._opts[opt.dest] = {'opt': opt, 'cli': cli}
        self._track_deprecated_opts(opt)
        return True
```

```python
    # 从group = self._get_group(group, autocreate=True) 跳转而来
    def _get_group(self, group_or_name, autocreate=False):
        """查找OptGroup对象.

        返回的OptGroup对象来自OptGroup对象的内部字典，
        它将是API用户有权访问的任何OptGroup对象的副本

        如果autocreate为True,且找不到该组，该group会被自动创建 
        如果group是OptGroup的实例，则将注册该实例，否则将创建OptGroup的新实例

        :param group_or_name: the group's name or the OptGroup object itself
        :param autocreate: whether to auto-create the group if it's not found
        :raises: NoSuchGroupError
        """
         
        group = group_or_name if isinstance(group_or_name, OptGroup) else None
        group_name = group.name if group else group_or_name

        if group_name not in self._groups:
            if not autocreate:
                raise NoSuchGroupError(group_name)

            self.register_group(group or OptGroup(name=group_name))

        return self._groups[group_name]
```

```python
    # 从return group._register_opt(opt, cli)跳转而来。这里也就是直接填充属性了
    # 记住，opt还是对象
    def _register_opt(self, opt, cli=False):
        """Add an opt to this group.

        :param opt: an Opt object
        :param cli: whether this is a CLI option
        :returns: False if previously registered, True otherwise
        :raises: DuplicateOptError if a naming conflict is detected
        """
         
        if _is_opt_registered(self._opts, opt):
            return False

        self._opts[opt.dest] = {'opt': opt, 'cli': cli}

        return True
```

![selg.\_opts&#x683C;&#x5F0F;](../.gitbook/assets/image%20%2854%29.png)

最后注册`deprecated_opts`到default组中

> 没提供group，默认注册到了default中

```python
# conf.register_opts(deprecated_opts)
步骤类似上面，不分析了
此时
api.register_opts(CONF) 执行完成了
```

`nova/conf/__init__.py`最后一条代码如下

```python
'''注册多个命令行 opts，忽略分析

/usr/local/bin/nova-compute --config-file /etc/nova/nova.conf
    --remote_debug-host <IP address where the debugger is running>
    --remote_debug-port <port> it's listening on>. 
'''
remote_debug.register_cli_opts(CONF)
```

此时，CONF对象初始化完成后，就已有许多的初始化属性了

## 3、main\(\)

```python
def main():
    config.parse_args(sys.argv)
    logging.setup(CONF, "nova")
    utils.monkey_patch()
    objects.register_all()
    if 'osapi_compute' in CONF.enabled_apis:
        # NOTE(mriedem): This is needed for caching the nova-compute service
        # version.
        objects.Service.enable_min_version_cache()
    log = logging.getLogger(__name__)

    gmr.TextGuruMeditation.setup_autorun(version)

    launcher = service.process_launcher()  # 启 worker
    started = 0
    for api in CONF.enabled_apis:
        should_use_ssl = api in CONF.enabled_ssl_apis
        try:
            server = service.WSGIService(api, use_ssl=should_use_ssl) # 返回了一个 WSGIService 对象
            launcher.launch_service(server, workers=server.workers or 1)
            started += 1
        except exception.PasteAppNotFound as ex:
            log.warning("%s. ``enabled_apis`` includes bad values. "
                        "Fix to remove this warning.", ex)

    if started == 0:
        log.error('No APIs were started. '
                  'Check the enabled_apis config option.')
        sys.exit(1)

    launcher.wait()
```

### 3.1、main函数 +40

```python
config.parse_args(sys.argv)
```

跳到了nova/config.py

```python
from oslo_log import log
from oslo_utils import importutils

from nova.common import config
import nova.conf
from nova.db.sqlalchemy import api as sqlalchemy_api
from nova import rpc
from nova import version

profiler = importutils.try_import('osprofiler.opts')


CONF = nova.conf.CONF


def parse_args(argv, default_config_files=None, configure_db=True,
               init_rpc=True):
    log.register_options(CONF)
    # We use the oslo.log default log levels which includes suds=INFO
    # and add only the extra levels that Nova needs
    if CONF.glance.debug:
        extra_default_log_levels = ['glanceclient=DEBUG']
    else:
        extra_default_log_levels = ['glanceclient=WARN']
    log.set_defaults(default_log_levels=log.get_default_log_levels() +
                     extra_default_log_levels)
    rpc.set_defaults(control_exchange='nova')
    if profiler:
        profiler.set_defaults(CONF)
    config.set_middleware_defaults()

    CONF(argv[1:],
         project='nova',
         version=version.version_string(),
         default_config_files=default_config_files)

    if init_rpc:
        rpc.init(CONF)

    if configure_db:
        sqlalchemy_api.configure(CONF)
```

```python
# nova/config.py +10

#osprofiler是一个请求跟踪库，opts中有它的所有选项，
#这里导入是为了，注册这些选项

profiler = importutils.try_import('osprofiler.opts')

#try_import最终调用了如下方法，
#__import__是动态加载模块
#返回的 sys.modules[import_str]，是module类型的对象，可以直接调用其方法

def import_module(import_str):
    __import__(import_str)
    return sys.modules[import_str]
```

```python
# nova/config.py +35
# 这里注册了 log 相关的选项，默认是INFO级别
log.register_options(CONF)

# 并且最终 globale了 CONF
_CONF = None

def _store_global_conf(conf):
    global _CONF
    _CONF = conf
```

```python
# nova/config.py +38
# 这里判断了 glance是否开启了debug，默认是没开的
    if CONF.glance.debug:
        extra_default_log_levels = ['glanceclient=DEBUG']
    else:
        extra_default_log_levels = ['glanceclient=WARN']
```

```python
# nova/config.py +42
# 设置了oslo.log 的 configuration options 的默认值
log.set_defaults(default_log_levels=log.get_default_log_levels() +
                     extra_default_log_levels)
                     
# 设置 默认的exchange 为 nova
rpc.set_defaults(control_exchange='nova')

# 设置了 osdporfiler的默认参数
if profiler:
    profiler.set_defaults(CONF)
  
# 设置 middleware的默认值
config.set_middleware_defaults()
```

```python
# nova/config.py +49
# 这里调用了 oslo_config/cfg.py 的 __call__方法
# 解析命令行传入的参数和配置文件
    CONF(argv[1:],
         project='nova',
         version=version.version_string(),
         default_config_files=default_config_files)
```

```python
# nova/config.py +54

# 初始化了消息队列的一些东西
    if init_rpc:
        rpc.init(CONF)

# 应用了conf.database和conf.api_database的配置
# 新增了监听器
    if configure_db:
        sqlalchemy_api.configure(CONF)
```

### 3.2、main函数 +41

```python
# Setup logging for the current application
    logging.setup(CONF, "nova")
    
# 此功能为指定模块中的所有功能patch装饰器。    
    utils.monkey_patch()

# 导入 nova/objects/ 下的所有函数    
    objects.register_all()

#这里是初始化了两个变量
#cls._MIN_VERSION_CACHE = {}
#cls._SERVICE_VERSION_CACHING = True
    if 'osapi_compute' in CONF.enabled_apis:
        objects.Service.enable_min_version_cache()

#创建了一个log对象
log = logging.getLogger(__name__)

#此方法将Guru Meditation Report设置为在收到给定信号(比如kill 服务)时
#自动转储到stderr或给定目录中的文件中 
#它还可以使用文件修改事件代替信号。
gmr.TextGuruMeditation.setup_autorun(version)
```

























