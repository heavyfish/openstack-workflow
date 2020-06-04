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
max_pool_size = 10
max_overflow = 20
idle_timeout = 60

[token]

# 撤消单个令牌的支持，通过token identifier 和 对各种令牌枚举操作（例如列出发行给特定用户的所有令牌）
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
# Keystone令牌也是令牌，因此较短的持续时间也将减少泄露的令牌对安全的潜在影响。
expiration = 21600

[cache]
backend = oslo_cache.memcache_pool
enabled = True
memcache_servers = 172.19.132.2:11211,172.19.132.3:11211,172.19.132.4:11211
expiration_time = 10800
memcache_dead_retry = 3600
memcache_socket_timeout = 1
memcache_pool_maxsize = 100

[cors]
allowed_origin = http://172.19.132.40:3000

[memcache]
dead_retry = 3600
socket_timeout = 1

[keystone_authtoken]
memcache_pool_dead_retry = 3600
memcache_pool_socket_timeout = 1
```



