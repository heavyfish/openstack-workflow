# RPC、REST、RESTful

## 1、简介

RPC：一种软件架构概念/API设计风格；

REST：一种软件架构概念/API设计风格；

RESTful：如果满足REST约束条件和原则的架构，就被称为是RESTful架构。就像URL都是URI\(统一资源标识\)的表现形式一样，RESTful是符合REST原则的表现形式。

不管是RPC还是RESTful，它们的传输层协议都是基于TCP/UDP的。而在应用层目前RESTful最常用的是HTTP协议，RPC用的是【JSON-RPC/gRPC/自定制】协议。

RPC和RESTful的主要区别在于应用层协议和架构上。

架构上的不同导致了 双方在接口的设计方式上，传输的Header+Body上，函数的调用上存在着区别

## 2、RPC

什么是 RPC 。RPC 是 `Remote Procedure Call` ，字面意思是远程过程调用。但是说白了就是从本地机器去执行服务器上的一个函数。所以说 RPC 指的是一类日常的操作，是个很宽泛的概念。

如果宽泛而言，HTTP 请求本身也是一种 RPC 的形式。因为HTTP 请求也可以从本地发一个信号到服务器，服务器上执行某个函数，然后返回一些信息给客户端。

![](../.gitbook/assets/image%20%2840%29.png)

如果放窄而言，可将 HTTP 请求和 RPC 作为平行概念来对比。说HTTP 好比普通话，而 RPC 是各地方言。这里的 RPC 此时指的是开发者自己手写的某种 RPC 形式，所以才能跟 HTTP 形成平行关系。

![](../.gitbook/assets/image%20%2845%29.png)

我们在谈到RPC时，一般都是将其放窄而讲的。上图是RPC的架构模式。

### 2.1、RPC模式

RPC 模式分为三层：

* **RPCRuntime** 负责最底层的网络传输；
* **Stub** 处理客户端和服务端约定好的语法、语义的封装和解封装，这些调用远程的细节都被这两层搞定了；
* **用户和服务器**这层就只要负责处理业务逻辑，调用本地 Stub 就可以调用远程

具体而言：

1. 客户端\(Client\)：服务调用方 
2. 客户端存根\(Client Stub\)：存放服务端地址信息，将客户端的请求参数打包成网络消息，再通过网络发送给服务方 
3. 服务端存根\(Server Stub\)：接受客户端发送过来的消息并解包，再调用本地服务
4.  服务端\(Server\)：真正的服务提供者

更具体的实现步骤：

1. 服务调用方（client）\(客户端\)以本地调用方式调用服务；
2. client stub接收到调用后负责将方法、参数等组装成能够进行网络传输的消息体；在Java里就是序列化的过程 
3.  client stub找到服务地址，并将消息通过网络发送到服务端； 
4. server stub收到消息后进行解码,在Java里就是反序列化的过程； 
5. server stub根据解码结果调用本地的服务； 
6. 本地服务执行处理逻辑； 
7. 本地服务将结果返回给server stub； 
8. server stub将返回结果打包成消息，Java里的序列化； 
9. server stub将打包后的消息通过网络并发送至消费方 
10. client stub接收到消息，并进行解码, Java里的反序列化； 
11. 服务调用方（client）得到最终结果。

RPC框架的目标就是把2-10步封装起来，把调用、编码/解码的过程封装起来，让用户像调用本地服务一样的调用远程服务

## 3、RESTful

REST即表述性状态传递（Representational State Transfer，简称REST），是一种软件架构风格。 

REST通过HTTP协议定义的通用动词方法\(GET、PUT、DELETE、POST\) ，以URI对网络资源进行唯一标识，响应端根据请求端的不同需求，通过无状态通信，对其请求的资源进行表述。

我们可以通过具体理解Representational State Transfer的意思，来明白RESTful的架构

### 3.1、资源（Resources）

REST的名称"表现层状态转化"中，省略了主语。"表现层"其实指的是"资源"（Resources）的"表现层"。

**所谓"资源"，就是网络上的一个实体，或者说是网络上的一个具体信息。**它可以是一段文本、一张图片、一首歌曲、一种服务，总之就是一个具体的实在。你可以用一个URI（统一资源定位符）指向它，每种资源对应一个特定的URI。要获取这个资源，访问它的URI就可以，因此URI就成了每一个资源的地址或独一无二的识别符。

所谓"上网"，就是与互联网上一系列的"资源"互动，调用它的URI

### 3.2、表现层（Representation）

"资源"是一种信息实体，它可以有多种外在表现形式。**我们把"资源"具体呈现出来的形式，叫做它的"表现层"（Representation）。**

比如，文本可以用txt格式表现，也可以用HTML格式、XML格式、JSON格式表现，甚至可以采用二进制格式；图片可以用JPG格式表现，也可以用PNG格式表现。

URI只代表资源的实体，不代表它的形式。严格地说，有些网址最后的".html"后缀名是不必要的，因为这个后缀名表示格式，属于"表现层"范畴，而URI应该只代表"资源"的位置。它的具体表现形式，应该在HTTP请求的头信息中用Accept和Content-Type字段指定，这两个字段才是对"表现层"的描述。

### 3.3、状态转化（State Transfer）

访问一个网站，就代表了客户端和服务器的一个互动过程。在这个过程中，势必涉及到数据和状态的变化。

互联网通信协议HTTP协议，是一个无状态协议。这意味着，所有的状态都保存在服务器端。因此，**如果客户端想要操作服务器，必须通过某种手段，让服务器端发生"状态转化"（State Transfer）。而这种转化是建立在表现层之上的，所以就是"表现层状态转化"。**

客户端用到的手段，只能是HTTP协议。具体来说，就是HTTP协议里面，四个表示操作方式的动词：GET、POST、PUT、DELETE。它们分别对应四种基本操作：**GET用来获取资源，POST用来新建资源（也可以用于更新资源），PUT用来更新资源，DELETE用来删除资源。**

### **3.4、无状态**

**无状态** REST设计风格要求Server无状态，无状**态并不等于**不保存用户的状态，而是指服务器不保存请求状态（会话信息），客户端必须每次都**带上自己的状态**去请求服务器，如果确实要维持用户的状态，也应由客户端负责，例如：在服务端上通过Cookie保存Token，之后的请求中都带上Token，而这个Token就保存有了用户的状态（如登录信息）。这里需要注意的是：

* 通过Session保存状态**不是REST设计风格**，因为Session是将状态信息（用户信息、过期时间等）保存在了服务器上，比如用户登录成功后，会将Session信息保存在服务器，然后返回个SessionID给客户端并且将SessionID保存在Cookies中,之后的请求客户端都会通过Cookies传递SessionID给服务器，服务器根据客户端传来的SessionID去匹配之前保存的Session状态信息，所以这个状态是保存在服务器上的，是靠服务器维持的，所以不是REST设计风格。
* 通过Token保存状态**是REST设计风格**，因为状态信息（用户信息、过期时间等）都是保存在Token中，而Token又是保存在客户端中（如Cookies），比如用户登录成功后，服务器会返回一个Token（包含了用户信息、过期时间等）给服务端，服务端将Token保存在Cookies中，之后的请求客户端都会取出Token放到Request Headers中传给服务器，服务器验证Token的有效性即可。

### **3.4、综述**

综合上面的解释，我们总结一下什么是RESTful架构：

（1）每一个URI代表一种资源；

（2）客户端和服务器之间，传递这种资源的某种表现层；

（3）客户端通过四个HTTP动词，对服务器端资源进行操作，实现"表现层状态转化"。

一句话总结：后端将资源发布为URI，前端通过URI访问资源，并通过HTTP动词表示要对资源进行的操作

## 4、异同点

RPC和RESTful的主要区别在于通信协议和架构上。

### 4.1、序列化和通信协议

接口调用通常包含两个部分，序列化和通信协议。常见的序列化协议包括json、xml、hession、protobuf、thrift、text、bytes等；通信协议比较流行的是http、soap、websockect。

RPC通常基于TCP实现，常用框架例如netty，Dubbo

RESTful通常采用http+JSON实现。 

JSON-RPC是指通信协议采用二进制方式，而不是http，序列化采用JSON的形式

不管是RPC还是RESTful，它们的传输层协议都是基于TCP/UDP的。而在应用层目前RESTful最常用的是HTTP协议，RPC用的是【JSON-RPC/gRPC/自定制】协议。

RPC不使用HTTP协议的原因是因为：HTTP协议在传输过程中有太多的Header浪费了空间

### 4.2、API设计

架构上的不同导致了 双方在接口的设计方式上，函数的调用上存在着区别。

以一个订餐API为例：

![](../.gitbook/assets/image%20%2859%29.png)

RPC**面向过程（或者说是action）**，只发送 GET 和 POST 请求。GET用来查询信息，其他情况下一律用POST。请求参数是**动词**，直接描述动作本身。

RESTful**面向资源**，使用 POST、DELETE、PUT、GET 请求，分别对应增、删、改、查操作。请求参数是**名词**，这个名词就是“增删改查”想要操作的对象。

### 4.3、传输的Message

在看一个向用户”发送消息“的示例：

```http
# RPC
POST /SendUserMessage HTTP/1.1
Host: api.example.com
Content-Type: application/json

{"userId": 501, "message": "Hello!"}
```

```http
# RESTful
POST /users/501/messages HTTP/1.1
Host: api.example.com
Content-Type: application/json

{"message": "Hello!"}
```

RPC永远调用的是一个函数，并将相关参数放在message中传输

RESTful在调用URI时，就已经知道了自己要操作的对象（资源）

## 5、选择建议

* RESTful API：主要用在为第三方提供调用自家系统的一种途径。
* RPC：主要用在自家系统之间的互相调用，即实现系统的分布式

具体的还是根据业务场景来设计。

## **6、其余概念**

### **6.1、**RPC 与 Dubbo

RPC虽然经常被当作API设计风格进行讨论，但它更多的是一种通信方式或概念（RPC不是协议！）。而Dubbo是实现了RPC概念的框架，使得调用远程服务就像调用本地方法一样，用过Dubbo的人肯定明白。所以RPC与Dubbo是实现与被实现的关系。我们常说Dubbo是阿里巴巴开源的RPC框架，就是这个道理。

这里需要提到的一点，就是**Dubbo框架**与**Dubbo协议**，如果只说Dubbo一词，其实是有歧义的，因为Dubbo可以是框架也可以是协议。Dubbo框架默认使用的是Dubbo协议，这个协议是阿里巴巴自己实现的一种应用层协议，传输层还是TCP。所以Dubbo协议与HTTP、FTP，SMTP这些应用层协议是并列的概念。除了默认的Dubbo协议，Dubbo框架还支持RMI、Hessian、HTTP等协议。

### 6.2、Dubbo、RMI、HTTP

如果将Dubbo与RMI、HTTP并列在一起，那么这个Dubbo就是指Dubbo框架默认的Dubbo协议。

RMI，这是一种古老的协议，在JDK1.1中就被实现了，我不知道那个时候RPC概念有没有被提出，但是RMI其实就是一种RPC的实现，比Dubbo要早得多。RMI的通信方式是把Java对象序列化为二进制格式，接收方收到以后再进行反序列化，所以局限性就是用RMI通信的系统都必须是用Java语言编写的

## 7、继续深入

[比对了json-RPC和RESTful](https://cloud.tencent.com/developer/article/1196179)

[RESTful API 最佳实践](http://www.ruanyifeng.com/blog/2018/10/restful-api-best-practices.html)

