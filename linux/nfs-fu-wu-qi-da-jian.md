---
description: 在CentOS 7上搭建NFS服务器
---

# NFS服务器搭建

## 1、环境准备

1、安装NFS

```text
yum install nfs-utils
```

2、关闭selinux和iptables

3、启动NFS服务

```text
systemctl start rpcbind
systemctl start nfs
```

4、创建共享目录

```text
mkdir /home/nfs
```

5、修改/etc/exports

```text
/home/nfs *(rw,async,no_root_squash)
/home/nfs 172.16.143.0/24(rw,async,anonuid=0,anongid=0)
```

```text
#使配置生效
exportfs -rv
```

6、确认共享目录已被分享出来

```text
showmount -e localhost
```

7、在其他服务器上挂载

```text
mount -t nfs -o nolock <IP>:/home/nfs <本地目录>
```

## 2、共享参数

下面是一些NFS共享的常用参数： 

```text
ro 只读访问 
rw 读写访问 
sync 所有数据在请求时写入共享 
async NFS在写入数据前可以相应请求 
secure NFS通过1024以下的安全TCP/IP端口发送 
insecure NFS通过1024以上的端口发送 
wdelay 如果多个用户要写入NFS目录，则归组写入（默认） 
nowdelay 如果多个用户要写入NFS目录，则立即写入，当使用async时，无需此设置。 
Hide 在NFS共享目录中不共享其子目录 
no_hide 共享NFS目录的子目录 
subtree_check 如果共享/usr/bin之类的子目录时，强制NFS检查父目录的权限（默认） 
no_subtree_check 和上面相对，不检查父目录权限 
all_squash 共享文件的UID和GID映射匿名用户anonymous，适合公用目录。 
no_all_squash 保留共享文件的UID和GID（默认） 
root_squash root用户的所有请求映射成如anonymous用户一样的权限（默认） 
no_root_squash root用户具有根目录的完全管理访问权限 
anonuid=xxx 指定NFS服务器/etc/passwd文件中匿名用户的UID 
例如可以编辑/etc/exports为： 
/tmp (rw,no_root_squash) 
/home/public 192.168.0.(rw) (ro) 
/home/test 192.168.0.100(rw) 
/home/linux *.the9.com(rw,all_squash,anonuid=40,anongid=40) 
```

