# 文件上传漏洞

如果处理不当，上传的文件可能会构成很大的风险。远程攻击者可以发送带有特殊构造的文件名或者有mime类型的multipart/form-data POST请求来执行任意代码。

## Summary

* [Tools](#tools)
* [Exploits](#exploits)
	* [PHP Extension](#php-extension)
	* [Other extensions](#other-extensions)
	* [Upload tricks](#upload-tricks)
	* [Picture upload with LFI](#picture-upload-with-lfi)
	* [Configuration Files](#configuration-files)
	* [CVE - Image Tragik](#cve---image-tragik)
* [References](#references)


## Tools

- [Fuxploider](https://github.com/almandin/fuxploider)

## Exploits

### PHP Extension

php后缀名

```powershell
.php
.php3
.php4
.php5
.php7

很少有人知道的后缀名
.pht
.phar
.phpt
.pgif
.phtml
.phtm

双重后缀名
.jpeg.php
.jpg.php
.png.php
```

### Other extensions

其他后缀名

```powershell
asp : .asp, .aspx
perl: .pl, .pm, .cgi, .lib
jsp : .jsp, .jspx, .jsw, .jsv, .jspf
Coldfusion: .cfm, .cfml, .cfc, .dbm
```

### Upload tricks

上传欺骗方式

- Null字节填充(eg:shell.php%00.gif, shell.php%00.png)，很好的攻击`pathinfo()`
- Mime type, 修改 `Content-Type : application/x-php` 或者 `Content-Type : application/octet-stream` 为`Content-Type : image/gif`
- 

### Picture upload with LFI

在图片中隐写PHP代码。上传图片并使用本地文件包含可以执行这些代码，可以用以下命令执行相应代码`curl 'http://localhost/test.php?0=system' --data "1='ls'"`.

- 图片元数据，将payload可以注入到元数据的注释标记内。
- Picture Resize, hide the payload within the compression algorithm in order to bypass a resize. Also defeating `getimagesize()` and `imagecreatefromgif()`.
- 图片调整大小，将payload藏在压缩算法中来绕过调整大小。也可以攻击`getimagesize()` 和 `imagecreatefromgif()`.

### Configuration Files

- .htaccess
- web.config
- httpd.conf
- \_\_init\_\_.py


### CVE - Image Tragik

```powershell
HTTP Request
Reverse Shell
Touch command
```

## References

* Bulletproof Jpegs Generator - Damien "virtualabs" Cauquil
* [BookFresh Tricky File Upload Bypass to RCE, NOV 29, 2014 - AHMED ABOUL-ELA](https://secgeek.net/bookfresh-vulnerability/)
* [Encoding Web Shells in PNG IDAT chunks, 04-06-2012, phil](https://www.idontplaydarts.com/2012/06/encoding-web-shells-in-png-idat-chunks/)
* [La PNG qui se prenait pour du PHP, 23 février 2014](https://phil242.wordpress.com/2014/02/23/la-png-qui-se-prenait-pour-du-php/)
* [File Upload restrictions bypass - Haboob Team](https://www.exploit-db.com/docs/english/45074-file-upload-restrictions-bypass.pdf)