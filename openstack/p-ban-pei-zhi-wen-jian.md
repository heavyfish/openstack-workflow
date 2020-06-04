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
connection = mysql+pymysql://keystone:Password@172.19.132.40:3306/keystone
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

# glance show image时，显示images的真实存储位置
show_image_direct_url = True
# glance show image时，返回images的所有存储位置，Pike及之后被移除
show_multiple_locations = True

# 当从service catalog中寻找cinder时，要匹配的信息
cinder_catalog_info = volume:cinder:internalURL

# ＃表示要使用的消息传递驱动程序及其完整配置的URL
transport_url = rabbit://openstack:Password@172.19.132.2:5672
# 用于连接数据库的SQLAlchemy连接字符串
connection = mysql+pymysql://glance:Password@172.19.132.40:3306/glance
max_retries = -1
max_pool_size = 10
max_overflow = 20
idle_timeout = 60

[keystone_authtoken]
# 完整的“public”身份API端点。S版本被移除，后续使用www_authenticate_uri=
auth_uri = http://172.19.132.40:5000
# 完整的“admin”身份API端点
auth_url = http://172.19.132.40:35357
auth_type = password
project_domain_id = default
user_domain_id = default
project_name = service
username = glance
password = 5bgBFrigaZryHEa53RADpPBorCAMW39qXkWcbqDL

# token数据被加密后仔仔cache中进行身份验证
memcache_security_strategy = ENCRYPT
# 此字符串用于密钥派生
memcache_secret_key = 4FlGmUgmcUrzB1KrJnye7V86iAtQq1quvp7jgJqN
memcached_servers = 172.19.132.2:11211,172.19.132.3:11211,172.19.132.4:11211
memcache_pool_dead_retry = 3600
memcache_pool_socket_timeout = 1

[paste_deploy]
# Deployment flavor to use in the server application pipeline
flavor = keystone

[glance_store]
default_store = rbd
stores = file,http,rbd,cinder
rbd_store_user = glance
rbd_store_pool = images
# image会分块为此大小的对象（单位：M）,值应该为2的幂次方
rbd_store_chunk_size = 8

[oslo_middleware]
enable_proxy_headers_parsing = True

[oslo_messaging_notifications]
# 用于处理发送 notifications 的driver。
# 可能的值messaging，messagingv2, routing, log, test, noop (multi valued)
driver = messagingv2
```

## 3、nova

```python
[DEFAULT]
debug = False
log_dir = /var/log/kolla/nova

# 此目录用于存储Nova的内部状态。 从中派生的各种其他配置选项都使用它。 
# 在某些情况下（例如，迁移），使用在多个计算主机之间共享的存储位置（例如，通过NFS）是有意义的。
# 除非选项``instances_path``被覆盖，否则该目录可能会变得很大。
state_path = /var/lib/nova

# The OpenStack API service 监听的IP
osapi_compute_listen = 172.19.132.2
# The OpenStack API service 监听的端口
osapi_compute_listen_port = 8774
# OpenStack API service 进程数. 默认为可用的CPU数量
osapi_compute_workers = 5

# metadat API 相关的
metadata_workers = 5
metadata_listen = 172.19.132.2
metadata_listen_port = 8775

allow_resize_to_same_host = true

# 定义用于控制虚拟化的驱动程序。
compute_driver = libvirt.LibvirtDriver

# 主机用于连接管理网络的IP地址
my_ip = 172.19.132.2

# 此选项启用定期的compute.instance.exists通知。 
# 必须将每个计算节点配置为生成系统使用情况数据。 
# 这些通知由OpenStack Telemetry服务使用。
instance_usage_audit = True
instance_usage_audit_period = hour

transport_url = rabbit://openstack:Pass@172.19.132.2:5672

# 在 config drive 强制注入
# 当此选项设置为true时，默认情况下将强制启用config drive，
# 否则用户仍可以通过REST API 或 image元数据属性启用配置驱动器。
force_config_drive = true

# 自上次报告up以来，可容忍无报告的最大时长
service_down_time = 120

# 是否删除未使用的base image
remove_unused_base_images = False

# 两次image cache manager之间等待的秒数。
image_cache_manager_interval = 0

# 此选项指定是否启动物理节点重启前之前正在运行的客户机。 
# 它确保每次计算节点启动或重新启动时，Nova计算节点上的所有实例都恢复其状态。
resume_guests_state_on_host_boot = True

# Seconds to wait for a response from a call. (integer value)
rpc_response_timeout = 300

# 等待Neutron VIF插入事件消息到达前的超时秒数
vif_plugging_timeout = 10
# Determine if instance should boot or fail on VIF plugging timeout.
vif_plugging_is_fatal = False

# 不推荐使用了，最好在[vnc]中加入enabled = true
vnc_enabled = true

[api]
use_forwarded_for = true

[conductor]
workers = 5

[vnc]
# noVNC控制台代理应绑定的IP地址。
novncproxy_host = 172.19.132.2
novncproxy_port = 6080
vncserver_listen = 172.19.132.2
# VNC控制台代理的私有，内部IP地址或主机名
vncserver_proxyclient_address = 172.19.132.2
# Public address of noVNC VNC console proxy.
novncproxy_base_url = http://172.19.132.40:6080/vnc_auto.html

[oslo_middleware]
enable_proxy_headers_parsing = True

[oslo_concurrency]
# 用于锁文件的目录。 
# 为了安全起见，指定目录只能由 运行 需要锁的进程 的用户写入。 
# 默认为环境变量OSLO_LOCK_PATH。 如果使用外部锁，则必须设置一个锁路径
lock_path = /var/lib/nova/tmp

[glance]
# 可用于nova的glance api服务器端点列表
api_servers = http://172.19.132.40:9292
# uploading / downloading image的尝试次数，0表示不重试
num_retries = 3
debug = False

[cinder]
# 在服务目录中查找cinder时要匹配的信息
catalog_info = volumev3:cinderv3:internalURL
os_region_name = RegionOne


[neutron]
# 此选项指定用于连接Neutron的URL
url = http://172.19.132.40:9696

# 此选项包含用于验证到neutron metadata的代理请求
# 为了被使用，'X-Metadata-Provider-Signature'标头必须在请求中提供
metadata_proxy_shared_secret = AlBIz8otjoshFQQ51oA2xeOLKMIbbYi6NqO7qZdu

# 设置为True时，此选项表示Neutron将使用proxy metadata请求并解析实例ID。 
# 否则，必须在“ X-Instance-ID”标头中将实例ID传递到元数据请求。
service_metadata_proxy = true

# Authentication URL
auth_url = http://172.19.132.40:35357
auth_type = password
project_domain_name = Default
user_domain_id = default
project_name = service
username = neutron
password = YFONWGgttKAsLSWo1GffIrTZtDVeF483wOsBVKox

[database]
connection = mysql+pymysql://nova:j5vuakfweENL2sTXYDdV6PIn0dVFJeLWaDialKQ8@172.19.132.40:3306/nova
max_pool_size = 10
max_overflow = 20
max_retries = -1
idle_timeout = 60

[api_database]
# Nova API数据库是一个单独的数据库，用于跨cells使用的信息
connection = mysql+pymysql://nova_api:NxF8kr3ydSntTU2UGzzxgB94Rc5XNhobz0228Dxg@172.19.132.40:3306/nova_api
max_retries = -1
max_overflow = 10
max_pool_size = 120
pool_timeout = 30
idle_timeout = 60

[cache]
backend = oslo_cache.memcache_pool
enabled = True
memcache_servers = 172.19.132.2:11211,172.19.132.3:11211,172.19.132.4:11211
expiration_time = 10800
memcache_dead_retry = 3600
memcache_socket_timeout = 1
memcache_pool_maxsize = 100

[keystone_authtoken]
auth_uri = http://172.19.132.40:5000
auth_url = http://172.19.132.40:35357
auth_type = password
project_domain_id = default
user_domain_id = default
project_name = service
username = nova
password = kH4EhHDUFV5P7UBVnZskUzRArcoSSa4vF8PYd4zP
memcache_security_strategy = ENCRYPT
memcache_secret_key = 4FlGmUgmcUrzB1KrJnye7V86iAtQq1quvp7jgJqN
memcached_servers = 172.19.132.2:11211,172.19.132.3:11211,172.19.132.4:11211
memcache_pool_dead_retry = 3600
memcache_pool_socket_timeout = 1

[libvirt]
# 配置了libvirt hypervisor相关的选项
# 几乎所有的libvirt配置选项都受``virt_type``配置的影响

# 覆盖所选虚拟化类型的默认libvirt URI。
connection_uri = qemu+tcp://172.19.132.2/system

# VM Images format
images_type = rbd
images_rbd_pool = vms
images_rbd_ceph_conf = /etc/ceph/ceph.conf
rbd_user = nova
disk_cachemodes = network=writeback
hw_disk_discard = unmap
rbd_secret_uuid = 5d4fbaf7-6eef-4ae9-908b-f7476cca0dd8

# libvirt应该采用的虚拟化类型
virt_type = kvm
inject_partition = -1
inject_password = True
cpu_mode = host-model

[upgrade_levels]
compute = auto

[oslo_messaging_notifications]
driver = messagingv2
topics = notifications

[privsep_entrypoint]
helper_command = sudo nova-rootwrap /etc/nova/rootwrap.conf privsep-helper --config-file /etc/nova/nova.conf

[guestfs]
debug = False

[wsgi]
api_paste_config = /etc/nova/api-paste.ini

[scheduler]
max_attempts = 5
discover_hosts_in_cells_interval = 60

[placement]
auth_type = password
auth_url = http://172.19.132.40:35357
username = placement
password = arZhoy83cccmjbtPaIcBiJ3ZILgQT6Reabzk17yN
user_domain_name = Default
project_name = service
project_domain_name = Default
os_region_name = RegionOne
os_interface = internal

[notifications]
notify_on_state_change = vm_and_task_state
notification_format = unversioned

[memcache]
dead_retry = 3600
socket_timeout = 1

[serial_console]
enabled = false

[filter_scheduler]
host_subset_size = 10
max_io_ops_per_host = 10
enabled_filters = RetryFilter,AvailabilityZoneFilter,ComputeFilter,ComputeCapabilitiesFilter,ImagePropertiesFilter,ServerGroupAntiAffinityFilter,ServerGroupAffinityFilter,AggregateCoreFilter,AggregateDiskFilter,DifferentHostFilter,SameHostFilter
```

