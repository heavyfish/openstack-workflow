# P版配置文件

## 1、keytsone

```python
[DEFAULT]
debug = False
log_file = /var/log/kolla/keystone/keystone.log

# Log output to standard error. This option is ignored if log_config_append is
# set. (boolean value)
use_stderr = True

[oslo_middleware]

# Whether the application is behind a proxy or not. This determines if the
# middleware should parse the headers or not. (boolean value)
enable_proxy_headers_parsing = True

[database]
connection = mysql+pymysql://keystone:qfSYNSMxqxsRiaUd9EHumQ3aPqTJ0mdelMvmsl8q@172.19.132.40:3306/keystone
# Set to -1 to specify an infinite retry count
max_retries = -1
# Maximum number of SQL connections to keep open in a pool
max_pool_size = 10
max_overflow = 20

# 闲置连接的超时时间
idle_timeout = 60

[token]

# toggle 撤消单个令牌的支持，通过token identifier 和 对各种令牌枚举操作（例如列出发行给特定用户的所有令牌）
# 这些操作用于确定要撤消的令牌列表。 如果您使用的是`kvs`` [revoke]驱动程序，请不要禁用此选项。
revoke_by_id = False

# Entry point for the token provider in the `keystone.token.provider`
# namespace. The token provider controls the token construction, validation,
# and revocation operations. Keystone includes `fernet` and `uuid` token
# providers. `uuid` tokens must be persisted (using the backend specified in
# the `[token] driver` option), but do not require any extra configuration or
# setup. `fernet` tokens do not need to be persisted at all, but require that
# you run `keystone-manage fernet_setup` (also see the `keystone-manage
# fernet_rotate` command). (string value)
provider = uuid

# 令牌应保持有效的时间（以秒为单位）。
# 大幅降低该值可能会破坏涉及多个服务以进行协调的“长时间运行”操作，并且将迫使用户更频繁地通过keystone校正进行身份验证。
# 急剧增加此值将增加[token]驱动程序的负载，因为更多的令牌将同时有效。
# Keystone令牌也是Bearer token，因此较短的持续时间也将减少泄露的令牌对安全的潜在影响。
expiration = 21600

[cache]
# Dogpile.cache后端模块。建议使用Memcache或Redis
#（dogpile.cache.redis）用于生产部署。对于基于事件的
# 或高度线程化的服务器，建议使用 Memcache with pooling （oslo_cache.memcache_pool）
backend = oslo_cache.memcache_pool

# Global toggle for caching
enabled = True
memcache_servers = 172.19.132.2:11211,172.19.132.3:11211,172.19.132.4:11211

# dogpile.cache区域中任何缓存的项目的默认TTL（以秒为单位）。
expiration_time = 10800

# memcache被认为dead后，等待再次尝试的时间
memcache_dead_retry = 3600
# 每次调用服务的超时时间（以秒为单位）
memcache_socket_timeout = 1
# 与每个Memcached服务器的最大打开连接总数。
memcache_pool_maxsize = 100

[cors]
# 此资源可被跨域访问
allowed_origin = http://172.19.132.40:3000

[memcache]
# memcached 服务被认为dead后，再次尝试访问的时间
dead_retry = 3600
# 每次调用服务的超时时间（以秒为单位）.键值存储系统使用它。（整数值）
socket_timeout = 1

[keystone_authtoken]
# memcache被认为dead后，等待再次尝试的时间
memcache_pool_dead_retry = 3600
# 每次调用服务的超时时间（以秒为单位）
memcache_pool_socket_timeout = 1
```

## 2、glance

### 2.1、glance-api

```python
[DEFAULT]
debug = False
log_file = /var/log/kolla/glance/glance-api.log

# 绑定到glance 服务的地址
bind_host = 172.19.132.2

# glance 服务监听的端口
bind_port = 9292

# glance服务启动时的线程数
workers = 5

# 仓库主机地址
registry_host = 172.19.132.40

# glance show image时，显示images的
show_image_direct_url = True

show_multiple_locations = True
cinder_catalog_info = volume:cinder:internalURL
transport_url = rabbit://openstack:NQgqidpgJXZ9XRDr0Sgd64yiRbEPYoDWc24XPXS8@172.19.132.2:5672,openstack:NQgqidpgJXZ9XRDr0Sgd64yiRbEPYoDWc24XPXS8@172.19.132.3:5672,openstack:NQgqidpgJXZ9XRDr0Sgd64yiRbEPYoDWc24XPXS8@172.19.132.4:5672

[database]
connection = mysql+pymysql://glance:LXPfczPdjl7HrX6kfF3UfLb1t1Cdp3gpaGGrYXEM@172.19.132.40:3306/glance
max_retries = -1
max_pool_size = 10
max_overflow = 20
idle_timeout = 60

[keystone_authtoken]
auth_uri = http://172.19.132.40:5000
auth_url = http://172.19.132.40:35357
auth_type = password
project_domain_id = default
user_domain_id = default
project_name = service
username = glance
password = 5bgBFrigaZryHEa53RADpPBorCAMW39qXkWcbqDL
memcache_security_strategy = ENCRYPT
memcache_secret_key = 4FlGmUgmcUrzB1KrJnye7V86iAtQq1quvp7jgJqN
memcached_servers = 172.19.132.2:11211,172.19.132.3:11211,172.19.132.4:11211
memcache_pool_dead_retry = 3600
memcache_pool_socket_timeout = 1

[paste_deploy]
flavor = keystone

[glance_store]
default_store = rbd
stores = file,http,rbd,cinder
rbd_store_user = glance
rbd_store_pool = images
rbd_store_chunk_size = 8

[oslo_middleware]
enable_proxy_headers_parsing = True

[oslo_messaging_notifications]
driver = messagingv2

[cache]
expiration_time = 10800
memcache_dead_retry = 3600
memcache_socket_timeout = 1
memcache_pool_maxsize = 100

[memcache]
dead_retry = 3600
socket_timeout = 1
```

