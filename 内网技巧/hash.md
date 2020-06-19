## windows hash

> windows hash 主要分为nt hash , ntlm hash , net ntlm hash ,nt hash容易破解，在现在测试中已经几乎见不到了。

## 目录

* [windows hash 分类]()
* [破解 ntlm hash]()
* [kerberos流程]()
* [hash传递]()
* [票据传递]()
* [ntlm中继]()

## windows hash 分类

* LM Hash：

    LM Hash是一种较古老的Hash，在LAN Manager协议中使用，非常容易通过暴力破解获取明文凭据。Vista以前的Windows OS使用它，Vista之后的版本默认禁用了LM协议，但某些情况下还是可以使用。

* NTLM Hash

    Vista以上现代操作系统使用的Hash。通常意义上的NTLM Hash指存储在SAM数据库及NTDS数据库中对密码进行摘要计算后的结果，这类Hash可以直接用于PtH，并且通常存在于lsass进程中，便于SSP使用

* Net-NTLM Hash

    Net-NTLM Hash用于网络身份认证（例如ntlm认证中），目前分为两个版本：Net-NTLMv1/Net-NTLMv2

    通常我们使用Responder等工具获取到的就是NetNTLM，这类hash并不能直接用来PtH，但可以通过暴力破解来获取明文密码

## 破解ntlm hash 

## kerberos流程

## hash传递

## 票据传递

## ntlm中继

