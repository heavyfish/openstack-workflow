# mariadb galera 集群恢复

## 前言

本文主要内容为：kolla部署的容器化mairadb galera集群的恢复及相关基础知识。

## 简介

Galera Cluster的所有节点是对等关系，每个节点均支持写入。

#### 官方给出特性如下：

* 普通列表项目真正的多主集群，Active-Active架构。
* 同步复制，没有复制延迟。
* 多线程复制。
* 没有主从切换操作，无需使用虚IP。
* 热备份，单个节点故障期间不会影响数据库业务。
* 支持节点自动加入，无需手动拷贝数据。
* 支持InnoDB存储引擎。
* 对应用程序透明，原生MySQL接口。
* 无需做读写分离。

#### 查看服务状态

查看容器日志：`docker logs mariadb`  
 查看服务日志：`cat /var/log/kolla/mariadb/mariadb.log`  
 登陆数据库输入： `show global status like "wsrep_%";`  
 数据库状态表主要查看3个值：

1. wsrep\_cluster\_size: 集群节点数量
2. wsrep\_cluster\_status: 集群状态
3. wsrep\_local\_state\_comment: 当前节点状态

**wsrep\_cluster\_status :**

任何非Primary值都代表该集群不可用

**wsrep\_local\_state\_comment 状态对照表：**

| 状态 | 状态说明 |
| :--- | :--- |
| Open： | 节点启动成功，尝试连接到集群 |
| Primary： | 节点已处于集群中，在新节点加入时，选取donor进行数据库同步时会产生的状态 |
| Joiner： | 节点处于等待接收或正在接收同步文件的状态 |
| Joined： | 节点完成数据同步，但还有部分数据不是最新的，在追赶与集群数据一致的状态 |
| Synced： | 节点正常提供服务的状态，表示当前节点数据状态与集群数据状态是一致的 |
| Donor： | 表示该节点被选为Donor节点，正在为新加进来的节点进行全量数据同步，此时该节点对客户端不提供服务 |

**SST**

英文全称为State Snapshot Transfer，即状态快照迁移：通过从一个节点到另一个节点迁移完整的数据拷贝（全量拷贝）。当一个新的节点加入到集群中，新的节点从集群中已有节点进行数据同步，开始进行状态快照迁移。

**IST**

英文全称为Increamental State Transfer，即增量状态迁移：集群一个节点通过识别新加入的节点缺失的事务操作，将该操作发送，而并不像SST那样的全量数据拷贝。最常见情况就是该节点之前已经存在于该集群，只是关机重启了，重新加入该集群会使用IST进行同步。

**grastate.dat**

默认路径为：/var/lib/docker/volumes/mariadb/\_data/grastate.dat  
 可以通过该文件查看到该节点记录的uuid和seqno，也就是上面说的GTID，当节点正常退出Galera集群时，会将GTID的值更新到该文件中。在断电的情况下，所有节点的seqno值可能都相同，此时需根 gvwstate.dat判断启动节点

**gvwstate.dat**

默认路径为：/var/lib/docker/volumes/mariadb/\_data/grastate.dat  
 此文件保存了集群状态信息

## kolla-ansible 部署原理

占位

## kolla-ansible 恢复脚本原理

占位

## 手动恢复通用流程

1. 关闭所有节点mariadb：`docker stop mariadb`
2. 选择启动节点

* 对比所有节点`cat /var/lib/docker/volumes/mariadb/_data/grastate.dat`中seqno值，若该值在所有节点中存在唯一得最大值，则该节点选为启动节点
* 若seqno最大值相同得节点有多个，则 `cat /var/lib/docker/volumes/mariadb/_data/gvwstate.dat`文件中view\_id和my\_uuid相等得节点选为启动节点。
* 手动关闭所有服务器后，gvwstate.dat 文件会自动删除。若grastate.dat文件 seqno最大值不唯一，则在seqno最大的节点中随便选取一个节点作为启动节点。

1. 在启动节点上修改配置文件，`vim /etc/kolla/mariadb/galera.cnf` ，注释原有wsrep\_cluster\_address。新增 `wsrep_cluster_address = gcomm://`（表示新集群），保存退出。
2. 在启动节点上启动容器，`docker start mariadb`
3. 进入启动节点查看集群状态，注意替换数据库密码：

* `docker exec mariadb mysql -u root -p<MYSQL_PASSWORD> -e 'show global status like "wsrep_%";' | grep wsrep_cluster_size` 结果应当为 1
* `docker exec mariadb mysql -u root -p<MYSQL_PASSWORD> -e 'show global status like "wsrep_%";' | grep wsrep_local_state_comment` 结果应当为 Synced

1. 待启动节点启动完成以后，再依次启动其它节点（不要一次性启动所有）。

* 启动节点执行 `docker exec mariadb mysql -u root -p<MYSQL_PASSWORD> -e 'show global status like "wsrep_%";' | grep wsrep_local_state_comment` 结果应该为 donor。
* 耐心等待启动节点完成数据传输，直到 `docker exec mariadb mysql -u root -p<MYSQL_PASSWORD> -e 'show global status like "wsrep_%";' | grep wsrep_local_state_comment` 结果为 Synced
* 再启动其它节点；在任意已加入集群的节点上执行 `docker exec mariadb mysql -u root -p<MYSQL_PASSWORD> -e 'show global status like "wsrep_%";' | grep wsrep_cluster_size` 可以看到当前集群的节点数。在所有节点执行 `docker exec mariadb mysql -u root -p<MYSQL_PASSWORD> -e 'show global status like "wsrep_%";' | grep wsrep_local_state_comment`，执行结果为donor时表示有新节点加入正在同步数据。等待所有节点执行 `docker exec mariadb mysql -u root -p<MYSQL_PASSWORD> -e 'show global status like "wsrep_%";' | grep wsrep_local_state_comment` 结果都为synced时表示 集群启动成功。

1. 待所有节点启动完成以后，回到启动节点，`vim /etc/kolla/mariadb/galera.cnf`，将wsrep\_cluster\_address改回原值。
2. 在启动节点上， `docker stop mariadb`\(注意：docker restart 可能会出问题，待更多测试\)
3. 在启动节点上，`docker start mariadb`
4. 随便进入一个节点查看集群状态

## kolla-ansible mariadb galera 恢复参考

#### 环境介绍

kolla-ansible部署三节点，机器断电重启时容器会自启

机房服务器断电，全部服务器都重启。mariadb 集群出问题  
修复方法：

1. 关闭所有control几点的mariadb 容器

   ```text
   # ansible -i multinode control -m shell -a “docker stop mariadb”
   ```

2. 确保所有contoller的/var/lib/docker/volumes/mariadb/\_data/grastate.dat 里面的safe\_to\_bootstrap 只有一个值为1

   ```text
   # ansible -i multinode all -m shell -a “cat /var/lib/docker/volumes/mariadb/_data/grastate.dat
   ```

3. 使用mariadb\_recovery 修复集群

   ```text
   # kolla-ansible -i multinode mairadb_recovery  -e mariadb_recover_inventory_name=< safe_to_bootstrap=1的那一台>
   ```

## FAQ

#### Unknown/unsupported storage

```text
2018-11-07 17:34:25 140218605119680 [ERROR] InnoDB: space [header](http://www.php.net/header) page consists of zero bytes in data [file](http://www.php.net/file) ./ibdata1
2018-11-07 17:34:25 140218605119680 [ERROR] InnoDB: Could not open or create the [system](http://www.php.net/system) tablespace. If you tried to add new data files to the [system](http://www.php.net/system) tablespace, and it failed here, you should now edit innodb_data_file_path in my.cnf back to what it was, and remove the new ibdata files InnoDB created in this failed attempt. InnoDB only wrote those files full of zeros, but did not yet use them in any way. But be careful: do not remove old data files which contain your precious data!
2018-11-07 17:34:25 140218605119680 [ERROR] Plugin 'InnoDB' init function returned error.
2018-11-07 17:34:25 140218605119680 [ERROR] Plugin 'InnoDB' registration as a STORAGE ENGINE failed.
2018-11-07 17:34:25 140218605119680 [Note] Plugin 'FEEDBACK' is disabled.
2018-11-07 17:34:25 140218605119680 [ERROR] Could not open [mysql](http://www.php.net/mysql).plugin table. Some plugins may be not loaded
2018-11-07 17:34:25 140218605119680 [ERROR] Unknown/unsupported storage engine: innodb
2018-11-07 17:34:25 140218605119680 [ERROR] Aborting'<\br>
```

删除ib文件，rm –rf /var/lib/docker/volumes/mariadb/\_data/ib\_log\* （注意，若不生效，删除ib\*），重启mariadb

#### Recovered position 00000000-0000-0000-0000-000000000000:-1

```text
181107 17:34:59 mysqld_safe Starting mysqld daemon with databases from /var/lib/[mysql](http://www.php.net/mysql)/
181107 17:34:59 mysqld_safe WSREP: Running position recovery with --log_error='/var/lib/mysql//wsrep_recovery.UR5wHx' --pid-[file](http://www.php.net/file)='/var/lib/mysql//kycloud1-recover.pid'
181107 17:36:00 mysqld_safe WSREP: Recovered position 00000000-0000-0000-0000-000000000000:-1
181107 17:38:41 mysqld_safe mysqld from pid [file](http://www.php.net/file) /var/lib/[mysql](http://www.php.net/mysql)/mariadb.pid ended
```

此信息表示尝试SST传输，但是未成功，可能为集群信息不正确，如已启动节点：`vim /etc/kolla/mariadb/galera.cnf`中得wsrep\_cluster\_address未包含此节点。  
 **注意，此步骤可能花费较长时间，未出现ended 都不能确定是否失败。并且kolla自带恢复脚本可能因同步时间过长而报timeout的错误**

#### kolla恢复脚本出错01

日志输出如下信息：

```text
[ERROR] WSREP: failed to open backend connection:131: invalid UUID:00000000 (FATAL)
```

删除文件 rm /var/lib/docker/volumes/mariadb/\_data/gvwstate.dat, 继续执行恢复脚本

#### 使用kolla自带命令恢复失败后，某个节点的mariadb无法加入集群

**现象**：  
 此节点的mariadb启动后始终会建立新的集群，无法加入已启动的集群。  
 **原因**：kolla选取启动节点后重新run了容器，携带参数`BOOTSTRAP_ARGS=--wsrep-new-cluster`，此时恢复出现异常（timeout之类），恢复退出但是容器环境变量不正确，所以此容器每次重启都会建立新集群，也会产生恢复命令每次都会选择同一个节点作为启动节点的现象。  
 **解决方案**：

1.  `service docker stop` 没写错，停止docker服务。
2. 设置`/var/lib/docker/containers/<container_id>/config.v2.json`中的"`BOOTSTRAP_ARGS=`"\(置空即可，原值应当为`BOOTSTRAP_ARGS=--wsrep-new-cluster`\)
3. `service docker start`

## 参考链接

1. 官方文档（数据库集群恢复及相关状态对照）  [https://www.penguincomputing.com/documentation/scyld-cloud-manager/admin-guide/admin/galera/intro.html](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.penguincomputing.com%2Fdocumentation%2Fscyld-cloud-manager%2Fadmin-guide%2Fadmin%2Fgalera%2Fintro.html)
2. 官方文档（数据库集群状态）  [http://galeracluster.com/documentation-webpages/monitoringthecluster.html](https://links.jianshu.com/go?to=http%3A%2F%2Fgaleracluster.com%2Fdocumentation-webpages%2Fmonitoringthecluster.html)
3. 博客（数据库集群恢复）  [https://blog.csdn.net/zengxuewen2045/article/details/51868976](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fzengxuewen2045%2Farticle%2Fdetails%2F51868976)
4. 博客 （数据库集群恢复）  [https://www.cnblogs.com/luohaixian/p/9426359.html](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.cnblogs.com%2Fluohaixian%2Fp%2F9426359.html)

