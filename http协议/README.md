# HTTP协议 #

write:[@katherine](http://katherinehu.info/)

目录

* [HTTP协议简介](#HTTP协议简介)
* [TCP/IP协议族](#TCP/IP协议族)

    *  [TCP/IP通信传输流](#TCP/IP通信传输流)
        *  [与HTTP密切联系的协议：IP、TCP、DNS](#与HTTP密切联系的协议：IP、TCP、DNS)

* [HTTP/1.1](#HTTP/1.1)

  *  [HTTP协议的请求/响应步骤](#HTTP协议的请求/响应步骤)
  *  [HTTP的各种传输方法](#HTTP的各种传输方法)
  *  [返回结果的HTTP状态码](#返回结果的HTTP状态码)

* [HTTP报文](#HTTP报文)

  * [报文结构](#报文结构)

  * [HTTP首部字段](#HTTP首部字段)

    * [通用首部字段](#通用首部字段)

    * [请求首部字段](#请求首部字段)
    * [响应首部字段](#响应首部字段)
    * [实体首部字段](#实体首部字段)
    * [非HTTP/1.1首部字段](#非HTTP/1.1首部字段)

* [参考](#参考)

  



## HTTP协议简介 ##

HTTP（Hyper Text Transfer Protocol)<超文本传输协议>是一个简单的请求-响应协议，它通常运行在TCP之上。它指定了客户端可能发送给服务器什么样的消息以及得到什么样的响应。请求和响应消息的头以ASCII码形式给出；而消息内容则具有一个类似MIME的格式。这个简单模型是早期Web成功的有功之臣，因为它使得开发和部署是那么的直截了当。

万维网WWW（world wide web）发源于欧洲日内瓦量子物理实验室CERN，正是WWW技术的出现使得因特网得以超乎想象的速度迅猛发展。这项基于TCP/IP的技术在短短的十年时间内迅速成为已经发展了几十年的Internet上的规模最大的信息系统，它的成功归结于它的简单、实用。在WWW的背后有一系列的协议和标准支持它完成如此宏大的工作，这就是Web协议族，其中就包括HTTP超文本传输协议。

在1990年，HTTP就成为WWW的支撑协议。当时由其创始人WWW之父蒂姆·贝纳斯·李（TimBemers—Lee）提出，随后WWW联盟（WWW Consortium）成立，组织了IETF（Internet Engineering Task Force）小组进一步完善和发布HTTP协议。 

HTTP是应用层协议，同其他应用层协议一样，是为了实现某一类具体应用的协议，并由某一运行在用户空间的应用程序来实现其功能。HTTP是一种协议规范，这种规范记录在文档上，为真正通过HTTP协议进行通信的HTTP的实现程序。

HTTP协议是基于C/S架构进行通信的，而HTTP协议的服务器端实现程序有httpd、nginx等，其客户端的实现程序主要是Web浏览器，例如Firefox、InternetExplorer、Google chrome、Safari、Opera等，此外，客户端的命令行工具还有elink、curl等。Web服务是基于TCP的，因此为了能够随时响应客户端的请求，Web服务器需要监听在80/TCP端口。这客户端浏览器和Web服务器之间就可以通过HTTP协议进行通信了。

每个万维网站点都有一个服务器进程，它不断地监听TCP的80端口，以便发现是否有浏览器（即万维网客户）向它发出建立连接请求。一旦监听到建立连接请求，首先建立TCP连接，然后浏览器向万维网服务器发出浏览某个网页的请求，服务器返回所请求的网页作为响应。最后释放TCP连接。在浏览器和服务器之间的请求和响应的交互，必须按照规定的格式和遵循一定的规则。这些格式和规则就是超文本传送协议。

## TCP/IP协议族 ##

为了更好的了解HTTP，我们要先简单了解一下TCP/IP协议族。通常使用的网络都是在TCP/IP协议族的基础上运作的，而HTTP属于它内部的一个子集。TCP/IP按层次可以分为4层：应用层、传输层、网络层和数据链路层

```
应用层		  Telnet、FTP和Email等

运输层		  TCP和UDP

网络层		  IP、ICMP和GMP

链路层       设备驱动程序及接口卡
```



- 应用层：应用层决定了向用户提供应用服务时通信的活动

- 传输层：传输层对上层应用层，提供处于网络连接中的两台计算机之间的传输数据。

- 网络层：网络层用来处理在网络上流动的数据包。

- 数据链路层：用来处理连接网络的硬件部分。

  ### TCP/IP通信传输流 ###

  

```
            客户端							   服务器
            
应用层		HTTP客户端							HTTP客户端
			↓↑								   ↑↓
			↑↓								   ↑↓
传输层		  TCP								 TCP
			↑↓								   ↑↓
			↑↓								   ↑↓
网络层		  IP								 IP
			↑↓								   ↑↓
			↑↓								   ↑↓
链路层		  网络 							   网络
			 ↑_________________________________↑

```

利用TCP/IP进行网络通信时，发送端（客户端）从应用层向下走，接收端（服务器）则从链路层往上走。

举个栗子：

		客户周杰伦想看一下自己粉丝后援会的网页，于是他在电脑上进行操作，他作为发送端在应用层（HTTP协议）发出一个想看粉丝后援会官网主页页面的HTTP请求。接着，为了传输方便，在传输层（TCP协议）把从应用层收到的数据（称为HTTP请求报文）进行分割，并且在各个报文上打上标记序号及端口号后转发给网络层。在网络层（IP协议），增加作为通讯目的地的MAC地址后转发给链路层，这样发往网络的通信请求就准备完全了。

接收端的服务器在链路层接收到从客户端发送过来的数据，依次往上层发送数据。当传输到应用层，才算真正接收到由客户端发送过来的HTTP请求。

这边要注意：发送端在层于层之间传输数据时，每经过一层时必定会被打上一个该层所属的首部信息。在接收端层于层传输数据时，每经过一层时也会把对应的首部去掉。

### 与HTTP密切联系的协议：IP、TCP、DNS ###

IP协议位于网络层，几乎所有使用网络的系统都会用到IP协议，所以在我们学习的过程中，IP协议是需要掌握的。

TCP协议位于运输层，提供可靠的字节流服务。字节流服务（Byte Stream Service）是指，为了方便传输，将大块数据分割以报文段（segment）为单位的数据包进行管理。可靠的是指TCP可以确保数据能够达到目标，所以在TCP进行传输时会采用三次握手策略。目的是向对方确认数据包是否送达。过程中使用了TCP的标志（flag）——SYN(synchronize)和ACK（acknowledgement)。发送端首先会发送一个带有SYN标志的数据包给接收端，告诉它“标有SYN的数据包我已经发给你了哟~”。接收端收到这个数据包之后会回传一个标有SYN/ACK的数据包，以此告诉发送端“我已经收到你的数据包啦”。最后发送端再回传一个带ACK标志的数据包，表示"好哒，我已经知道你收到啦~"，“握手”结束。如果在握手过程中某个阶段没有得到回应，TCP协议会再次以相同的顺序发送相同的数据包。

DNS（Domain Name System)服务位于应用层。它提供域名到IP地址之间的解析服务。



## HTTP/1.1 ##

HTTP协议和TCP/IP协议族内的众多其他协议相同，用于客户端和服务器之间的通信，应用HTTP协议时，必定时一端担任客户端角色，另一端担任服务器端角色。HTTP协议规定，请求从客户端发出，最后服务器端响应该请求并返回。就是说，先从客户端开始建立通信，服务器端在没有接收到请求之前不会发送响应。

这是一个实例：

客户端发送请求

```
GET / HTTP/1.1
Host: www.baidu.com
```

服务器发送响应

```
HTTP/1.1 200 OK
Connection: keep-alive
Content-Length: 154
Content-Type: text/html
Date: Fri, 06 Mar 2020 03:19:41 GMT
Location: https://www.baidu.com/
```

客户端发给某个HTTP服务器端请求报文中的内容：

```
GET /index.html HTTP/1.1
Host: www.baidu.com
```

开头的GET表示请求访问服务器的类型，称为方法。随后的字符串"index.html"指明了请求访问的资源对象，也叫做请求URL。HTTP/1.1是HTTP的版本号，用来提示客户端使用的HTTP协议功能。

所以说，请求报文是由请求方法，请求URL，协议版本，可选的请求首部字段和内容实体构成的。

接收到请求的服务器，会将请求内容的处理结果以响应的形式返回。

```
HTTP/1.1 200 OK
Connection: keep-alive
Content-Length: 161
Content-Type: text/html
Date: Fri, 06 Mar 2020 04:22:37 GMT

<html>
...
```

开头的HTTP/1.1表示http的版本号，200OK表示请求的处理结果的状态码（status code)和原因短语（reason-phrase）。下面四行都称为响应首部字段，接着以空一行分隔，之后的内容成为资源实体的主体（entitybody）

注意：HTTP是一种不保存状态，即无状态（stateless）协议，它自身不对请求和相应之间的通信状态进行保存，每当有新的请求发送时，就会有对应的新响应产生。这是为了更快的处理大量事务，确保协议可伸缩。但是随着网络的不断发达，无状态协议就显得十分鸡肋，比如我们登录一个网站，输入了用户名和密码，然后我们跳转到这个网站的其他页面后也需要保持之前的登陆状态，所以后来就引入了cookie技术。

cookie技术举个简单的栗子：

福尔摩斯想看看华生写的博客，但是网站必须登陆后才能查看，于是福尔摩斯的客户端向服务器发出请求想查看这个网页，服务器生成了一个Cookie记住是向谁发送的，同时在响应中添加Cookie后返回。福尔摩斯想查看这个网站中的其他页面，但是由于他已经有了Cookie，所以他的客户端向服务器发送的第二次请求中已经添加了Cookie，服务器收到后会检查这个Cookie，然后它会知道”啊又是刚刚那个家伙啊“于是就直接响应，不需要第二次登陆。

### HTTP协议的请求/响应步骤 ###

客户端连接到web服务器

> 一个HTTP客户端,通常是浏览器,与Web服务器的HTTP端口(默认是80)建立一个TCP套接字连接。

发送HTTP请求

> 通过TCP套接字,客户端向Web服务器发送一个文本的请求报文,一个请求报文由请求行,请求头部,空行和请求体4个部分构成.

服务区接收解释请求并返回HTTP响应

> Web解析请求,定位请求资源.服务器将资源复本写到TCP套接字,由客户端获取.一个响应由状态行,响应 头,空行和响应数据4部分组成.

.释放连接TCP连接

> 若Connection模式为close,则服务器主动关闭TCP连接,客户端被动关闭TCP连接,释放TCP连接.若Connection为keepalive,则该连接会保持一段时间,该时间内可以持续使用该连接接收请求,做出响应

客户端浏览器解析HTML内容



花花也想看看自己粉丝后援会的主页，客户端就会发出请求“我想要看看http://xxidol.com/index.php/ 页面”这个时候客户端就会向DNS查询xxidol.com的ip地址，DNS会告诉客户端xxidol.com对应的IP地址是XXX.XXX.XXX.XXX。这个时候HTTP协议就生成一个请求报文“请给我http://xxidol.com/index.php/ 的页面资源” TCP协议会将这个报文按照序列号分为多个报文段，每个报文段都可靠的传给对方。然后IP协议会搜索对方的地址，一边中转，一边传送。然后接收方从对方接收到报文段，按序列号重组请求报文，HTTP协议会对Web服务器请求的内容进行处理知道“原来你想看/index.php/网页啊”。然后再把这个网页通过TCP/IP协议进行回传，这样花花就成功浏览这个网页。

### HTTP的各种传输方法 ###

1. GET：获取资源

   GET方法用来请求访问已被URL识别的资源。指定的资源经服务器端解析后返回响应内容

2. POST：传输实体主体

   POST的功能与GET很类似，但是POST的主要目的不是获取响应的主要内容。而且POST传输安全性更高一些。

3. PUT：传输文件

   PUT要求在请求报文的主体中包含文件内容，然后保存到请求URI指定的位置。但是HTTP/1.1中PUT方法自身不带验证机制，任何人都可以上传文件，所以安全性比较低。

4. HEAD：获得报文首部

   HEAD和GET一样，只是不返回报文主体部分，用于确定URI的有效性及资源更新的时间和日期等。

5. DELETE：删除文件

   DELETE按请求URI删除指定的资源，它本身也不带验证，所以安全性也比较低。

6. OPTIONS：询问支持方法

   OPTIONS方法用来查询针对请求URI指定的资源支持的方法。

7. TRACE：追踪路径

   TRACE方法是让Web服务器将之前的请求通信返回给客户端的方法。发送请求时，在Max-Forwards 首部字段中填入数值，每经过一个服务器端就将该数字减1，当数值刚好减到0时，就停止继续传输，最后接收到请求的服务器端则返回状态码200OK的响应。通过TRACE方法可以查询发送出去的请求是怎样被加工修改/篡改的。但是它容易引发XST（Cross-Site Tracing,跨站追踪）攻击，通常不会去使用

8. CONNECT：要求用隧道协议连接代理

   CONNECT方法要求在与代理服务器通信时建立隧道，实现用隧道协议进行TCP通信。主要使用SSL(Secure Sockets Layer,安全套接层)和TLS（Transport Layer Security，传输层安全）协议把通信内容加密后经网络隧道传输。

注意：方法名区分大小写注意要用大写字母！



### 返回结果的HTTP状态码 ###

- 2xx-成功

  200 OK  成功；

  201  Created 已创建；
  　　服务器已经创建了文档，Location头给出了它的URL。

  202  Accepted 接受；
  　　已经接受请求，但处理尚未完成。

  203 Non-Authoritative Information 非权威的信息；
  　　文档已经正常地返回，但一些应答头可能不正确，因为使用的是文档的拷贝，非权威性信息（HTTP 1.1新）。

  204 No Content 没有内容；

- 3xx-重定向

  300  Multiple Choices 多重选择；

  301  Moved Permanently 永久移动；
  　　客户请求的文档在其他地方，新的URL在Location头中给出，浏览器应该自动地访问新的URL。

  302  Found 发现；
  　  但新的URL应该被视为临时性的替代，而不是永久性的

  303 See Other 

  	表示由于请求对应的资源存在着另一个URL，应使用GET方法定向获取请求的资源

  304 Not Modified

  	表示客户端发送附带条件的请求时，服务器端允许请求访问资源，但因发生请求未满足条件的情况后，直接返回304 Not Modified。

  307 Temporary Redirect 临时重定向

- 4xx - 客户端错误

  400 - Bad Request 错误请求；
  　　请求出现语法错误。

  401 - Unauthorized 未授权；
  　  访问被拒绝，客户试图未经授权访问受密码保护的页面。

  403 - Forbidden 禁止访问；　　
      资源不可用。

  404 - Not Found 找不到；
  　　无法找到指定位置的资源。

- 5xx - 服务器错误
  　　服务器由于遇到错误而不能完成该请求。

  500 - Internal Server Error 服务器内部错误；
  　　服务器遇到了意料不到的情况，不能完成客户的请求。

  501 - Not Implemented 没有实现；
  　　服务器不支持实现请求所需要的功能，页眉值指定了未实现的配置。

  503 - Service Unavailable 服务不可用；
  　　服务器由于维护或者负载过重未能应答。

  504 - Gateway Timeout 网关超时；
  　　由作为代理或网关的服务器使用，表示不能及时地从远程服务器获得应答。

  505 - HTTP Version Not Supported http版本不支持；
  　　服务器不支持请求中所指明的HTTP版本。（HTTP 1.1新）

[本部分参考博客](https://www.cnblogs.com/xzsty/p/11452610.html)

[本部分参考博客](https://www.jianshu.com/p/7c8b4576e4bb)

## HTTP报文 ##

用于HTTP协议交互的信息称为HTTP报文，客户端的HTTP报文叫做请求报文，服务器端的叫做响应报文。HTTP报文本身是由多行数据构成的字符串文本。HTTP报文基本分为报文首部和报文主体两块。两者由空行划分，并不一定要有报文主体。

### 报文结构 ###

请求报文分为报文首部、空行、报文主体三个结构。请求报文的报文首部包括：请求行、请求首部字段、通用首部字段、实体首部字段和其他。

响应报文也分为报文首部、空行、报文主体三个结构。响应报文的报文首部包括：状态行、响应首部字段、通用首部字段、实体首部字段和其他。

一个实例：

请求报文：

```
POST /cloudquery.php HTTP/1.1			//请求行
User-Agent: Post_Multipart				          
Host: 221.130.199.34
Accept: */*
Proxy-Connection: Keep-Alive						各种首部字段
Pragma: no-cache
X-360-Cloud-Security-Desc: Scan Suspicious File
x-360-ver: 4
Content-Length: 690						         
------------------------------------------------------------------
空行

```

请求行：包含用于请求的方法（例子中为POST），请求URL（cloudquery.php)和HTTP版本(HTTP 1.1)



响应报文

```
HTTP/1.1 200 OK						//状态行
Server: nginx
Date: Sat, 07 Mar 2020 11:02:36 GMT       
Content-Type: application/octet-stream
Connection: close							 各种首部字段
Cache-Control: no-cache
pragma: no-cache
Content-Length: 161						
------------------------------------------------------------------
空行
------------------------------------------------------------------
报文主体

```

状态行：包含表明响应结果的而状态码（200），原因短语（OK），和HTTP版本（HTTP 1.1）

首部字段：包含表示请求和相应的各种条件和属性的各类首部。

其他：可能包含HTTP的RFC里没有定义的首部（Cookie等）

### HTTP首部字段 ###

HTTP协议的请求和响应报文中必定包含HTTP首部。首部的内容会包含客户端和服务器分别处理请求和响应所提供需要的信息。在报文众多的字段中，HTTP 首部字段包含的信息最为丰富，首部字段同时存在于请求和响应报文中，并涵盖HTTP报文相关的内容信息。

HTTP首部字段是由首部字段名和字段值构成的，中间由冒号分隔

```
首部字段名：字段值

```

比如

```
Content-Type: text/html
//Content-Type代表这个字段用来表示报文主体的对象类别，例子中为text/html
```

HTTP首部字段类型根据实际用途被分为以下4种类型：

通用首部字段（General Header Fields）：请求报文和响应报文两方都会使用的首部。

请求首部报文（Request Headers Fields）：从客户端向服务端发送请求报文时使用的首部。补充了请求的附加内容，客户端信息，响应内容相关优先级等信息。

响应首部字段（Response Header Fields）：从服务器端向客户端返回响应报文时使用的首部。补充了响应的附加内容，也会要求客户端附加额外的内容信息。

实体首部字段（Entity Header Fields）：针对请求报文和响应报文的实体部分使用的首部。补充了资源内容更新时间等与实体相关的信息。

HTTP/1.1规范定义了如下47种首部字段。我们会来分析一下这些字段

#### 通用首部字段 ####

| 首部字段名        | 说明                       |
| :---------------- | -------------------------- |
| Cache-Control     | 控制缓存的行为             |
| Connection        | 连接的管理                 |
| Date              | 创建报文的日期时间         |
| Pragma            | 报文指令                   |
| Trailer           | 报文末端的首部一览         |
| Transfer-Encoding | 指定报文主体的传输编码方式 |
| Upgrade           | 升级为其他协议             |
| Via               | 代理服务器的相关信息       |
| Warning           | 错误通知                   |

通用首部字段是指，请求报文和响应报文双方都会使用的首部。

##### 1、Cache-Control #####

通过首部字段Cache-Control的指令可以操作缓存的工作机制

Cache-Control 指令一览

可用的指令按请求和响应分类如下所示：

缓存请求指令：

| 指令                | 参数   | 说明                           |
| ------------------- | ------ | ------------------------------ |
| no-cache            | 无     | 强制向原服务器再次验证         |
| no-store            | 无     | 不缓存请求或响应的任何内容     |
| max-age = [ 秒]     | 必须   | 响应的最大Age值                |
| max-stale( = [ 秒]) | 可省略 | 接收已过期的响应               |
| min-fresh = [ 秒]   | 必需   | 期望在指定的时间内的响应仍有效 |
| no-transform        | 无     | 代理不可更改媒体类型           |
| only-if-cached      | 无     | 代理不可更改媒体类型           |
| cache-extension     | -      | 新指令标记（token）            |

```
1、Cache-Control：no-cache //为了防止从缓存中返回过期的资源
2、Cache-Control：no-store//暗示请求或响应中包含机密信息，所以该指令规定缓存不能在本地存储请求或响应的任意部分。
3、Cache-Control：max-age=604800（单位：秒）//当客户端发送的请求中包含这个指令时，如果判定缓存时间数值比指定时间的数值更小，那么客户端就接收缓存的资源。当值为0时，缓存服务器通常要将请求转发给源服务器。
4、Cache-Control：max-stale=3600（单位：秒）//可指示缓存资源，即使过期也照常接收。
5、Cache-Control：min-fresh=50（单位：秒）//要求缓存服务器返回至少还未过指定时间的资源缓存。
6、Cache-Control：no-transform//无论是在请求还是在响应中，缓存都不能改变实体主体的媒体类型。
7、Cache-Control：only-if-cached//客户端仅在缓存服务器缓存目标资源的情况下，才会要求其返回。
8、cache-extension用来标记token
```

缓存响应指令：

| 指令             | 参数   | 说明                                           |
| ---------------- | ------ | ---------------------------------------------- |
| public           | 无     | 可向任意方提供响应的缓存                       |
| private          | 可省略 | 仅向特定用户返回响应                           |
| no-cache         | 可省略 | 缓存前必需先确认其有效性                       |
| no-store         | 无     | 不缓存请求或响应的任何内容                     |
| no-transform     | 无     | 代理不可更改媒体类型                           |
| must-revalidate  | 无     | 可缓存但必须再向源服务器进行确认               |
| proxy-revalidate | 无     | 要求中间缓存服务器对缓存的响应有效性再进行确认 |
| max-age=[秒]     | 必需   | 响应的最大Age值                                |
| s-maxage=[秒]    | 必需   | 公共缓存服务器响应的最大Age值                  |
| cache-extension  | -      | 新指令标记（token）                            |

```
1、Cache-Control：public//其他用户也可以利用缓存
2、Cache-Control：private//响应只以特定的用户作为对象。
3、Cache-Control：no-cache//缓存服务器不能对资源进行缓存
4、Cache-Control：no-store//暗示请求或响应中包含机密信息，所以该指令规定缓存不能在本地存储请求或响应的任意部分。
5、Cache-Control：no-transform//无论是在请求还是在响应中，缓存都不能改变实体主体的媒体类型。
6、Cache-Control：must-revalidate//代理会向源服务器再次验证即将返回的响应缓存目前是否任然有效。
7、Cache-Control：proxy-revalidate//要求所有的缓存服务器在接收到客户端带有该指令的请求返回响应之前，必须再次验证缓存的有效性。
8、Cache-Control：max-age=604800（单位：秒）//当服务器返回的相应中包含max-age时，缓存服务器将不对资源的有效性再做确认，而max-age的数值代表资源保存为缓存的最长时间。
```

##### 2、Connection #####

Connection首部字段具备如下两个作用

1. 控制不再转发给代理的首部字段

   在客户端发送请求和服务器返回响应内，使用Connection首部字段可以控制不再转发代理的首部字段。

   ```
   Connection：不再转发的首部字段名
   ```

2. 管理持久连接

   HTTP/1.1版本的默认连接都是持久连接，所以客户端会在持久连接上连续发送请求。当服务器想明确断开连接时，则指定Connection首部字段的值为Close。

   ```
   Connection：close
   ```

##### 3、Data #####

首部字段Date表明创建HTTP报文的日期和时间。

```
Date: Sat, 07 Mar 2020 11:02:36 GMT 
```

##### 4、Pragma #####

Pragma是HTTP/1.1之前版本的历史遗留字段，仅作为与HTTP/1.0的向后兼容而定义。

规范定义的形式唯一，如下所示：

```
Pragma: no-cache
```

该首部字段属于通用首部字段，但只用在客户端发送的请求中。客户端会要求所有的中间服务器不返回缓存的资源。

所有的中间服务器如果都能以 HTTP/1.1 为基准，那直接采用 Cache-Control: no-cache 指定缓存的处理方式是最为理想的。但要整体掌握全部中间服务器使用的 HTTP 协议版本却是不现实的。因此，发送的请求会同时含有下面两个首部字段。

```
Cache-Control: no-cache
Pragma: no-cache
```

##### 5、 Trailer #####

首部字段Trailer会事先说明在报文主体后记录了哪些首部字段。

##### 6、Transfer-Encoding #####

规定传输报文主体时采用的编码方式。

##### 7、Upgrade #####

首部字段 Upgrade 用于检测 HTTP 协议及其他协议是否可使用更高的
版本进行通信，其参数值可以用来指定一个完全不同的通信协议。

##### 8、Via #####

使用首部字段Via是为了追踪客户端与服务器之间的请求和响应报文的传输路径。报文经过代理或网关时，会先在首部字段Via中附加该服务器的信息，再进行转发。

##### 9、Warning #####

该首部通常会告知用户一些与缓存相关的问题的警告。

```
warning：[警告码][警告的主机：端口号]"[警告内容]"（[日期时间]）
```

| 警告码 | 警告内容                                       | 说明                                                         |
| ------ | ---------------------------------------------- | ------------------------------------------------------------ |
| 110    | Response is stale(响应已过期)                  | 代理返回已过期的资源                                         |
| 111    | Revalidation failed(再验证失效)                | 代理再验证资源有效性时失败（服务器无法到达等原因）           |
| 112    | Disconnection operation(断开连接操作)          | 代理与互联网连接被故意切断                                   |
| 113    | Heuristic expiration(试探性过期)               | 响应的使用期超过24小时（有效缓存的设定时间大于24小时的情况下） |
| 199    | Miscellaneous warning(杂项警告)                | 任意的警告内容                                               |
| 214    | Transformation applied(使用了转换)             | 代理对内容编码或媒体类型等执行了某些处理时                   |
| 299    | Miscellaneous persistent warning(持久杂项警告) | 任意的警告内容                                               |



#### 请求首部字段 ####

| 首部字段名          | 说明                                          |
| ------------------- | --------------------------------------------- |
| Accept              | 用户代理可处理的媒体类型                      |
| Accept-Charset      | 优先的字符集                                  |
| Accept-Encoding     | 优先的内容编码                                |
| Accept-Language     | 优先的语言（自然语言）                        |
| Authorization       | Web认证信息                                   |
| Expect              | 期待服务器的特定行为                          |
| From                | 用户的电子邮箱地址                            |
| Host                | 请求资源所在服务器                            |
| If-Match            | 比较实体标记（ETag）                          |
| If-Modified-Since   | 比较资源的更新时间                            |
| If-None-Match       | 比较实体标记（与 If-Match 相反）              |
| If-Range            | 资源未更新时发送实体 Byte 的范围请求          |
| If-Unmodified-Since | 比较资源的更新时间（与If-Modified-Since相反） |
| Max-Forwards        | 最大传输逐跳数                                |
| Proxy-Authorization | 代理服务器要求客户端的认证信息                |
| Range               | 实体的字节范围请求                            |
| Referer             | 对请求中URI的原始获取方                       |
| TE                  | 传输编码的优先级                              |
| User-Agent          | HTTP客户端程序的信息                          |

##### 1、Accept #####

```
Accept:text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3
```

Accept 首部字段可通知服务器，用户代理能够处理的媒体类型及媒体类型的相对优先级。可使用 type/subtype 这种形式，一次指定多种媒体类型。

##### 2、Accept-Charset #####

```
Accept-Charset: iso-8859-5, unicode-1-1;q=0.8
```

Accept-Charset 首部字段可用来通知服务器用户代理支持的字符集及字符集的相对优先顺序。另外，可一次性指定多种字符集。与首部字段 Accept 相同的是可用权重 q 值来表示相对优先级。

该首部字段应用于内容协商机制的服务器驱动协商。

##### 3、Accept-Encoding #####

```
Accept-Encoding: gzip, deflate
```

Accept-Encoding 首部字段用来告知服务器用户代理支持的内容编码及内容编码的优先级顺序。可一次性指定多种内容编码。

##### 4、Accept-Language #####

```
Accept-Language: zh-cn,zh;q=0.7,en-us,en;q=0.3
```

首部字段 Accept-Language 用来告知服务器用户代理能够处理的自然语言集（指中文或英文等），以及自然语言集的相对优先级。可一次指定多种自然语言集。和 Accept 首部字段一样，按权重值 q 来表示相对优先级。

##### 5、Authorization #####

用来告知服务器，用户代理的认证信息（证书值）

##### 6、Expect #####

告知服务器，期望出现的某种特定行为。

##### 7、From #####

告知服务器使用用户代理的用户电子邮件地址。

##### 8、Host #####

告知服务器请求的资源所处的互联网主机名和端口号。Host首部字段在HTTP/1.1规范内是唯一一个必须被包含在请求内的首部字段。

##### 9、If-Match、If-Modified-Since 、If-None-Match 、If-Range 、If-Unmodified-Since #####

形如 If-xxx 这种样式的请求首部字段，都可称为条件请求。服务器接收到附带条件的请求后，只有判断指定条件为真时，才会执行请求。所以If-Modified-Since  、 If-None-Match  、 If-Range 、 If-Unmodified-Since都类似。

首部字段 If-Match，属附带条件之一，它会告知服务器匹配资源所用的实体标记（ETag）值。这时的服务器无法使用弱 ETag 值。服务器会比对 If-Match 的字段值和资源的 ETag 值，仅当两者一致时，才会执行请求。反之，则返回状态码 412 Precondition Failed 的响应。

If-Modified-Since会告知服务器若If-Modified-Since字段值早于资源的更新时间，则希望能处理该请求。

If-Range 告知服务器若指定的If-Range 字段值（ETag值或时间）和请求资源的ETag值或时间相一致时，则作为范围请求处理。反之则返回全体资源

 If-Unmodified-Since告知服务器，指定的请求资源只有在字段值内指定的日期时间之后，未发生更新的情况下，才能处理请求。

##### 10、Max-Forwards #####

发送包含该首部的请求时，该字段以十进制整数形式指定可经过的服务器最大数目。

##### 11、Proxy-Authorization #####

接收到从代理服务器发来的认证质询时，客户端会发送包含首部字段Proxy-Authorization的请求，以告知服务器认证所需要的信息。

##### 12、Range #####

可告知服务器资源的指定范围。

```
Range:bytes=5001-10000//请求获取从第5001字节-10000字节的资源
```

##### 13、Referer #####

首部字段 Referer 会告知服务器请求的原始资源的 URI。

```
Referer: http://172.30.1.34:4200/
```

##### 14、TE #####

告知服务器客户端能够处理响应的传输编码方式及相对优先级。

##### 15、User-Agent #####

首部字段 User-Agent 会将创建请求的浏览器和用户代理名称等信息传达给服务器。

```
User-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/76.0.3809.87 Safari/537.36
```



#### 响应首部字段 ####

| 首部字段名         | 说明                         |
| ------------------ | ---------------------------- |
| Accept-Ranges      | 是否接受字节范围请求         |
| Age                | 推算资源创建经过时间         |
| ETag               | 资源的匹配信息               |
| Location           | 令客户端重定向至指定URI      |
| Proxy-Authenticate | 代理服务器对客户端的认证信息 |
| Retry-After        | 对再次发起请求的时机要求     |
| Server             | HTTP服务器的安装信息         |
| Vary               | 代理服务器缓存的管理信息     |
| WWW-Authenticate   | 服务器对客户端的认证信息     |

##### 1、Accept-Ranges #####

告知客户端服务器是否能处理范围请求，以指定获取服务器端某个部分的资源。可处理范围请求时指定其为bytes，反之则指定为none

##### 2、 Age #####

首部字段 Age 能告知客户端，源服务器在多久前创建了响应。字段值的单位为秒。

##### 3、ETag #####

首部字段 ETag 能告知客户端实体标识。它是一种可将资源以字符串形式做唯一性标识的方式。服务器会为每份资源分配对应的 ETag值。

##### 4、Location #####

使用首部字段 Location 可以将响应接收方引导至某个与请求 URI 位置不同的资源。基本上，该字段会配合 3xx ：Redirection 的响应，提供重定向的URI。

##### 5、Proxy-Authenticate #####

会把由代理服务器所要求的认证信息发送给客户端。

##### 6、Retry-After #####

告知客户端应该在多久之后再次发送请求

##### 7、Server #####

告知客户端当前服务器上安装的HTTP服务器应用 程序的信息。

##### 8、Vary #####

可对缓存进行控制，源服务器会向代理服务器传达关于本地缓存使用方法的命令。

##### 9、WWW-Authenticate #####

用于HTTP访问认证。

#### 实体首部字段 ####

| 首部字段名       | 说明                         |
| ---------------- | ---------------------------- |
| Allow            | 资源可支持的HTTP方法         |
| Content-Encoding | 实体主体适用的编码方式       |
| Content-Language | 实体主体的自然语言           |
| Content-Length   | 实体主体的大小（单位：字节） |
| Content-Location | 替代对应资源的URI            |
| Content-MD5      | 实体主体的报文摘要           |
| Content-Range    | 实体主体的位置范围           |
| Content-Type     | 实体主体的媒体类型           |
| Expires          | 实体主体过期的日期时间       |
| Last-Modified    | 资源的最后修改日期时间       |

##### 1、Allow #####

通知客户端能够支持Request-URI指定资源的所有HTTP方法。

##### 2、Content-Encoding #####

首部字段 Content-Encoding 会告知客户端服务器对实体的主体部分选用的内容编码方式。内容编码是指在不丢失实体信息的前提下所进行的压缩。

##### 3、Content-Language #####

```
Content-Language: zh-CN
```

首部字段 Content-Language 会告知客户端，实体主体使用的自然语言（指中文或英文等语言）。

##### 4、Content-Length #####

```
Content-Length: 15000
```

首部字段 Content-Length 表明了实体主体部分的大小（单位是字节）。对实体主体进行内容编码传输时，不能再使用 Content-Length首部字段。

##### 5、Content-Location #####

给出与报文主体部分相对应的URI。

##### 6、Content-MD5 #####

是由一串有MD5算法生成的值，可以检查报文主体在传输过程中是否保持完整，以及确认传输到达。

##### 7、Content-Range #####

能告知客户端作为响应返回的实体的哪个部分符合范围请求。

##### 8、Content-Type #####

说明了实体主体内对象的媒体类型。和首部字段 Accept 一样，字段值用 type/subtype 形式赋值。

##### 9、Expires #####

将资源失效的日期告知客户端。

##### 10、Last-Modified #####

首部字段 Last-Modified 指明资源最终修改的时间。

#### 非HTTP\/1.1首部字段 ####

RFC2616中定义的以上47中首部字段并非我们在HTTP通信交互中使用到的首部字段，还有另外一些首部字段也很常用，这些非正式首部字段统一归纳在RFC4299 HTTP Header Field Registrations中。

##### 为Cookie服务的首部字段 #####

| 首部字段名 | 说明                           | 首部类型     |
| ---------- | ------------------------------ | ------------ |
| Set-Cookie | 开始状态管理所使用的Cookie信息 | 响应首部字段 |
| Cookie     | 服务器接收到的Cookie信息       | 请求首部字段 |

1、Set-Cookie

当服务器准备开始管理客户端的状态时会事先告知各种信息。表格中列举了Set-Cookie字段的属性

| 属性         | 说明                                                         |
| ------------ | ------------------------------------------------------------ |
| NAME=VALUE   | 赋予Cookie的名称和其值（必须项）                             |
| expires=DATE | cookie的有效期（若不明确指定则默认为浏览器关闭前为止）       |
| path=PATH    | 将服务器上的文件目录作为Cookie的适用对象（若不指定则默认为文档所在的文件目录） |
| domain=域名  | 作为Cookie适用对象的域名（若不指定则默认为创建Cookie的服务器域名） |
| secure       | 仅在HTTPS安全通信时才会发送Cookie                            |
| HttpOnly     | 加以限制，使Cookie不能被JavaScript脚本访问。                 |

2、Cookie

告知服务器，当客户端想获得HTTP状态管理支持时，就会在请求中包含从服务器接收到的Cookie。

##### X-Frame-Options #####

属于HTTP响应首部，用于控制网站内容在其他Web网站的Frame标签内的显示问题，主要目的是为了防止点击劫持攻击。

字段值：

DENY：拒绝

SAMEORIGIN：仅同源域名下的页面匹配时许可。

##### X-XSS-Protection #####

属于HTTP响应首部，时针对XSS攻击的一种策略，用于控制XSS防护机制的开关。

字段值：

0：将XSS过滤设置成无效状态

1：将XSS 过滤设置成有效状态

##### DNT #####

属于HTTP请求首部，其中DNT是Do Not Track的简称，意为拒绝个人信息被收集，是表示拒绝被精准广告追踪的一种方法。

字段值：

0：同意被追踪

1：拒绝被追踪

##### P3P #####

属于HTTP响应首部，通过利用P3P技术，可以让Web网站上的个人隐私变成一种仅供程序可理解的形式，以达到保护用户隐私的目的

有关P3P[的详细规范标准请参看](http://www.w3.org/TR/P3P)

[本部分参考](https://www.cnblogs.com/xzsty/p/11452610.html)



## 参考 ##

图解HTPP【日】上野宣著

[参考博客1](https://www.cnblogs.com/xzsty/p/11452610.html)

[参考博客2](https://www.cnblogs.com/xzsty/p/11452610.html)

[参考博客3](https://www.jianshu.com/p/7c8b4576e4bb)