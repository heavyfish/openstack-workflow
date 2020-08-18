# Linux 自建rpm仓库

## 1、Linux常用操作

### 1.1、自建rpm仓库

1、创建存放rpm包的目录

```text
mkdir /rpm/centos/6/os/x86_64
mkdir /rpm/centos/7/os/x86_64
```

2、下载安装包到其中

```text
yum install --downloadonly \
--downloaddir=/root/rpm <Package_name>
```

3、创建仓库元数据

```text
# 需要先安装 createrepo
yum install createrepo -y

createrepo <rpm_dir>
```

4、修改repo配置文件

```text
[my_base]
name=this is test repo
baseurl=file:///root/rpm
gpgcheck=0
```

[创建远程仓库](https://www.cnblogs.com/qiuhom-1874/p/11487456.html)



