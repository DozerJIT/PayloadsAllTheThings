# 常见的中间件和数据库

## 目录

[TOC]

## 中间件

中间件是什么？

中间件是一种独立的系统软件或服务程序，分布式应用软件借助这种软件在不同的技术之间共享资源。中间件位于客户机/ 服务器的操作系统之上，管理计算机资源和网络通讯。是连接两个独立应用程序或独立系统的软件。相连接的系统，即使它们具有不同的接口，但通过中间件相互之间仍能交换信息。

执行中间件的一个关键途径是信息传递。通过中间件，应用程序可以工作于多平台或 OS 环境。

中间件是介于操作系统和应用软件之间，为应用软件提供服务功能的软件，有消息中间件，交易中间件，应用服务器等。由于介于两种软件之间，所以，称为中间件。

### 1. Hadoop

Hadoop 是一个分布式计算平台，由 Java 语言开发，包含 Common、MapReduce 和 HDFS 三个核心部件（HDFS 和 MapReduce 是最核心的两个部件）。其中：

-   Common 为 Hadoop 的其他项目提供了一些常用工具，主要包括系统配置工具 Configuration、远程过程调用 RPC、序列化机制和 Hadoop 抽象文件系统等。
-   MapReduce 是处理海量数据的计算模型。
-   HDFS 用于存储海量数据，它具备高度容错性，能在低成本的通用硬件机器上稳定运行。

基于 Hadoop 平台衍生出来的开源项目主要有 Yarn、HBase、Hive、ZooKeeper、Avro、Sqoop、Mahout、Crossbow 等。

### 2. LVS

LVS 是 Linux Virtual Server 的首字母缩写，意为 Linux 虚拟服务器，即把许多台物理 Linux 计算机逻辑上整合成一台超级计算机，对用户来说感觉只有一台计算能力很强的服务器。

LVS 就是一个由软件实现的负载均衡器，工作在网络 OSI 的第四层（应用层），是中国人章嵩开发的，代码已经并入了 Linux 内核。

### 3. Linux-HA

Linux-HA 意为 Linux 高可用性项目，此项目具体包含如下几个组件。

| 名称           | 作用                                                         |
| -------------- | ------------------------------------------------------------ |
| Heartbeat      | 负责维护集群中各节点的信息及它们之间的心跳通信。             |
| Pacemaker      | 集群资源管理器，是核心组件，客户端通过 Pacemaker 来配置、管理并监控整个集群。此组件的社区网站为 [http：//clusterlabs.org/](http://clusterlabs.org/)。OpenStack 高可用性部署实例中一般都采用 Pacemaker 和 HAProxy。 |
| Resource Agent | 为用于控制服务启停、监控服务状态的脚本集合，本地资源管理器（LRM）调用这些脚本来启动、停止、监控各种集群资源。 |
| Cluster Glue   | 包含一套函数库和工具，在集群栈中，除集群消息传输（由 Heartbeat 承担）、集群资源管理（由 Pacemaker 承担）和资源代理（由 Resource Agent 承担）功能外，其他功能都由 Cluster Glue 来完成。它包含的两个主要部分是 LRM 和 Stonith，前者是本地资源管理器，后者的任务是隔离故障机器。 |

### 4. 静态网站服务器

#### 4.1 Nginx

Nginx (engine x) 是一个高性能的HTTP和反向代理web服务器，同时也提供了IMAP/POP3/SMTP服务。Nginx是由伊戈尔·赛索耶夫为俄罗斯访问量第二的Rambler.ru站点开发的，第一个公开版本0.1.0发布于2004年10月4日。

[Nginx安装教程](https://www.cnblogs.com/lywJ/p/10710361.html)

[Nginx常见问题及解决](https://www.jb51.net/article/130057.htm)

#### 4.2 Apache

Apache 是世界使用排名第一的Web服务器软件，它的跨平台性和安全性使它成为最流行的Web服务器之一。

Apache源于NCSAhttpd服务器，Apache取自“a patchy server”的读音，意思是充满补丁的服务器。

[Apache常见问题及解决](https://www.baidu.com/s?ie=utf-8&f=8&rsv_bp=1&tn=SE_zzsearchcode_shhzc78w&wd=Apache%E5%B8%B8%E8%A7%81%E9%97%AE%E9%A2%98&oq=Apache%25E5%25B8%25B8%25E8%25A7%2581%25E9%2597%25AE%25E9%25A2%2598&rsv_pq=9939abec00085975&rsv_t=bcb2WeNplKR8nksKbcs669hdq55VNfVk6puPvw%2BdB3IEFKuULgjhqpyVRIq0vidqafex%2BoTgBC4HfEBjIhY1&rqlang=cn&rsv_enter=0&rsv_dl=tb&rsv_sug3=28&rsv_sug4=4076)

### 5. 动态应用服务器

开源的动态应用服务器有 JBoss、Tomcat、Geronimo、JOnAS。

#### 5.1 JBoss

是一个基于J2EE的开放源代码的应用服务器。 JBoss代码遵循LGPL许可，可以在任何商业应用中免费使用。JBoss是一个管理EJB的容器和服务器，支持EJB 1.1、EJB 2.0和EJB3的规范。

JBoss核心服务不包括支持servlet/JSP的WEB容器，一般与Tomcat或Jetty绑定使用。

#### 5.3 Weblogic

WebLogic是美国Oracle公司出品的一个application server，一个基于JAVAEE架构的中间件，WebLogic是用于开发、集成、部署和管理大型分布式Web应用、网络应用和数据库应用的Java应用服务器。将Java的动态功能和Java Enterprise标准的安全性引入大型网络应用的开发、集成、部署和管理之中。

[Weblogic和Tomcat的区别](https://www.iteye.com/blog/javatea-2040944)

#### 5.4 Websphere

WebSphere 是 IBM 的软件平台。它包含了编写、运行和监视全天候的工业强度的随需应变 Web 应用程序和跨平台、跨产品解决方案所需要的整个中间件基础设施，如服务器、服务和工具。WebSphere 提供了可靠、灵活和健壮的软件。

Websphere修改配置文件不用像tomcat那样重起服务器。
Websphere会把项目打包成EAR文件，部署这个EAR文件，TOMCAT是WAR文件。
他们的共同之处是都是支持JSP的服务器软件。
不同之处：
Tomcat： 是Apache Group Jakarta小组开发的一个免费服务器软件，适合于嵌入Apache中使用，而且，它的源代码是可以免费获得的，不足之处是它的配置十分麻烦，而且有一些安全性的问题没有解决，初学者可以用它来调试JSP文件，但是用作商业应用的服务器就不太妥当了。
BEA WebLogic Sever：是一款十分强大的服务器软件，配置比较简单，而且对JSP的扩展十分强大，附带了数据库的JDBC驱动程序，支持JHTML，是目前市场占有率最高的服务器

#### 5.5 Tomcat

Tomcat 服务器是一个免费的开放源代码的Web 应用服务器，属于轻量级应用服务器，在中小型系统和并发访问用户不是很多的场合下被普遍使用，是开发和调试JSP 程序的首选。对于一个初学者来说，可以这样认为，当在一台机器上配置好Apache 服务器，可利用它响应HTML（标准通用标记语言下的一个应用）页面的访问请求。实际上Tomcat是Apache 服务器的扩展，但运行时它是独立运行的，所以当你运行tomcat 时，它实际上作为一个与Apache 独立的进程单独运行的。

[Tomcat服务器下载、安装、配置环境变量教程](https://blog.csdn.net/qq_40881680/article/details/83582484)

## 数据库

### 1. MySQL

MySQL是关系型数据库管理系统，由瑞典MySQL AB 公司开发，属于 Oracle 旗下产品。MySQL 是最流行的关系型数据库管理系统之一，在 WEB 应用方面，MySQL是最好的 RDBMS (Relational Database Management System，关系数据库管理系统) 应用软件之一。

[MySQL安装教程](https://www.cnblogs.com/wangzepu/p/12201288.html)

### 2. Sql Server

美国Microsoft公司推出的一种关系型数据库系统。SQL Server是一个可扩展的、高性能的、为分布式客户机/服务器计算所设计的数据库管理系统，实现了与WindowsNT的有机结合，提供了基于事务的企业级信息管理系统方案。

[SQL Server2012安装教程](https://blog.csdn.net/zxy13826134783/article/details/82932684)

### 3. SQLite

SQLite，是一款轻型的数据库，是遵守ACID的关系型数据库管理系统，它包含在一个相对小的C库中。它是D.RichardHipp建立的公有领域项目。它的设计目标是嵌入式的，而且已经在很多嵌入式产品中使用了它，它占用资源非常的低，在嵌入式设备中，可能只需要几百K的内存就够了。它能够支持Windows/Linux/Unix等等主流的操作系统，同时能够跟很多程序语言相结合，比如 Tcl、C#、PHP、Java等，还有ODBC接口，同样比起Mysql、PostgreSQL这两款开源的世界著名数据库管理系统来讲，它的处理速度比他们都快。SQLite第一个Alpha版本诞生于2000年5月。 至2019年已经有19个年头，SQLite也迎来了一个版本 SQLite 3已经发布。

### 4. MongoDb

MongoDB是一个基于分布式文件存储的数据库。由C++语言编写。旨在为WEB应用提供可扩展的高性能数据存储解决方案。
MongoDB是一个介于关系数据库和非关系数据库之间的产品，是非关系数据库当中功能最丰富，最像关系数据库的。它支持的数据结构非常松散，是类似json的bson格式，因此可以存储比较复杂的数据类型。Mongo最大的特点是它支持的查询语言非常强大，其语法有点类似于面向对象的查询语言，几乎可以实现类似关系数据库单表查询的绝大部分功能，而且还支持对数据建立索引。

### 5. Redis

Redis（Remote Dictionary Server )，即远程字典服务，是一个开源的使用ANSI C语言编写、支持网络、可基于内存亦可持久化的日志型、Key-Value数据库，并提供多种语言的API。从2010年3月15日起，Redis的开发工作由VMware主持。从2013年5月开始，Redis的开发由Pivotal赞助。

## 参考：

[https://www.cnblogs.com/qmfsun/p/4055627.html](https://www.cnblogs.com/qmfsun/p/4055627.html)

[http://c.biancheng.net/view/3860.html](http://c.biancheng.net/view/3860.html)

