# MySQL中的8个 character\_set 变量

本篇会简单介绍在 MySQL 中关于 8个 character\_set 变量的基本作用。  
　　  
使用下列SQL语句可以查看 MySQL中8个 character\_set 变量

```text
SHOW VARIABLES LIKE '%char%';
```

![](../.gitbook/assets/image%20%2863%29.png)

#### 8个 character\_set 变量： <a id="8&#x4E2A;-characterset-&#x53D8;&#x91CF;"></a>

　　**一、character\_set\_client**

　　**二、character\_set\_connection**

　　**三、character\_set\_database**

　　**四、character\_set\_filesystem**

　　**五、character\_set\_results**

　　**六、character\_set\_server**

　　**七、character\_set\_system**

　　**八、character\_sets\_dir**

### _一、character\_set\_client_ <a id="&#x4E00;charactersetclient-1"></a>

　　主要用来设置客户端使用的字符集。

### _二、character\_set\_connection_ <a id="&#x4E8C;charactersetconnection-1"></a>

　　主要用来设置连接数据库时的字符集，如果程序中没有指明连接数据库使用的字符集类型则按照这个字符集设置。

### _三、character\_set\_database_ <a id="&#x4E09;charactersetdatabase-1"></a>

　　主要用来设置默认创建数据库的编码格式，如果在创建数据库时没有设置编码格式，就按照这个格式设置。

### _四、character\_set\_filesystem_ <a id="&#x56DB;charactersetfilesystem-1"></a>

　　文件系统的编码格式，把操作系统上的文件名转化成此字符集，即把 character\_set\_client转换character\_set\_filesystem， 默认binary是不做任何转换的。

### _五、character\_set\_results_ <a id="&#x4E94;charactersetresults-1"></a>

　　数据库给客户端返回时使用的编码格式，如果没有指明，使用服务器默认的编码格式。

### _六、character\_set\_server_ <a id="&#x516D;charactersetserver-1"></a>

　　服务器安装时指定的默认编码格式，这个变量建议由系统自己管理，不要人为定义。

### _七、character\_set\_system_ <a id="&#x4E03;charactersetsystem-1"></a>

　　数据库系统使用的编码格式，这个值一直是utf8，不需要设置，它是为存储系统元数据的编码格式。

### _八、character\_sets\_dir_ <a id="&#x516B;charactersetsdir-1"></a>

　　这个变量是字符集安装的目录。

**在启动mysql后，我们只关注下列变量是否符合我们的要求**

* character\_set\_client
* character\_set\_connection
* character\_set\_database
* character\_set\_results
* character\_set\_server

**下列三个系统变量我们不需要关心，不会影响乱码等问题**

* character\_set\_filesystem
* character\_set\_system
* character\_sets\_dir

_**更改以上字符集直接 set character\_set\_XXX = “gbk”;（XXX是写以上的变量名）**_

**借助网上的一个完整的用户请求的字符集转换流程来更好的理解上述几个变量：**

1. mysql Server收到请求时将请求数据从 **character\_set\_client** 转换为 **character\_set\_connection**
2. 进行内部操作前将请求数据从 **character\_set\_connection** 转换为内部操作字符集,步骤如下 　　A. 使用每个数据字段的 **CHARACTER SET** 设定值； 　　B. 若上述值不存在，则使用对应数据表的字符集设定值 　　C. 若上述值不存在，则使用对应数据库的字符集设定（character\_set\_database）

　            D. 若上述值不存在，则使用 **character\_set\_server** 设定值。

   3. 最后将操作结果从内部操作字符集转换为 **character\_set\_results**

