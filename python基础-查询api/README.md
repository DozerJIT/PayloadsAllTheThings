# Python基础-查询api

> Python在Web安全中起到了很好的辅助作用，python原本也是一个可以开发Web app的语言，在对于一些比较复杂的情况可以用python进行辅助操作。

## 目录

* [语言基础](#语言基础)
* [requests库](#requests库)
	* [请求示例](#请求示例)
	* [请求方式](#请求方式)
	* [响应的内容](#响应的内容)
	* [POST发送json请求](#POST发送json请求)
	* [定制请求头和cookie信息](#定制请求头和cookie信息)
	* [响应](#响应)
	* [设置请求时间](#设置请求时间)
	* [会话对象](#会话对象)
	* [设置请求代理](#设置请求代理)
* [urllib库](#urllib库)
* [bs4库](#bs4库)
* [ip-api.com爬虫](#ip-api.com爬虫)
* [bgp.he.net爬虫](#bgp.he.net爬虫)
* [参考](#参考)

## 语言基础

请移步廖雪峰Python教程：https://www.liaoxuefeng.com/wiki/1016959663602400

## requests库

requests库是一个基于Apache2 许可的HTTP库，基本支持HTTP所有请求方式

### 请求示例

```python
import requests

req = requests.get('<URL>')	#不带参数的GET请求
req2 = requests.get(url='<URL>',params={'p1':'v1'})	#带参数的GET请求
```

### 请求方式

```python
import requests

requests.get('<URL>')
requests.post('<URL>')
requests.put('<URL>')
requests.delete('<URL>')
requests.head('<URL>')
requests.options('<URL>')
```

### 响应的内容

```python
r.url	#显示当前请求的url(含参数)
r.encoding	#获取当前的编码
r.encoding = 'utf-8'	#设置编码
r.content	#以字节形式返回，字节形式的响应体，会自动为你解码gzip和deflate压缩
r.headers	#以字典对象存储服务器响应头，字典键不区分大小写，若键不存在则返回None
r.status_code	#响应状态码
r.raw	#返回原始响应体，也就是urllib的response对象，使用r.raw.read()
r.ok	#bool值代表是否登陆成功
r.json()	#以json形式返回，得先确保内容为json格式不然会报错
r.raise_for_status()	#失败请求(非200响应)抛出异常
```

### POST发送json请求

```python
import requests
import json

r=requests.post('<URL>',data=json.dumps({"some":"value"}))
print(r.json())
```

### 定制请求头和cookie信息

```python
header = {'user-agent': 'my-app/0.0.1''}
cookie = {'key':'value'}
r = requests.get/post('your url',headers=header,cookies=cookie) 
data = {'some': 'data'}
headers = {'content-type': 'application/json',
           'User-Agent': 'Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:22.0) Gecko/20100101 Firefox/22.0'}
 
r = requests.post('https://api.github.com/some/endpoint', data=data, headers=headers)
print(r.text)
```

### 响应

```python
r.headers                                  #返回字典类型,头信息
r.requests.headers                         #返回发送到服务器的头信息
r.cookies                                  #返回cookie
r.history                                  #返回重定向信息,当然可以在请求是加上allow_redirects = false 阻止重定向
```

### 设置请求时间

```python
r = requests.get('<URL>',timeout=1)
```

### 会话对象

```python
session=requests.Session()
session.auth('username','password')
session.headers = {'key','value'}
r=session.get('<URL>')
```

### 设置请求代理

```python
proxies = {'http':'ip1','https':'ip2' }
requests.get('url',proxies=proxies)
```

本部分参考：https://www.cnblogs.com/lanyinhao/p/9634742.html

## urllib库

这部分参考：https://www.cnblogs.com/zhangxinqi/p/9170312.html

这个库也是一个HTTP库，用法与requests库类似，就不在赘述。

## bs4库

这部分参考：https://www.cnblogs.com/zhangxinqi/p/9218395.html

## ip-api.com爬虫

ip-api WEB API:http://ip-api.com/json/\<URL\>?lang=zh-CN

这个只需要使用的requests库就能解决问题

```python
#!/usr/bin/python3 
import requests
import json

def api(ip):
    js=requests.get("http://ip-api.com/json/"+ip+"?lang=zh-CN")
    js.encoding='utf-8'
    dic=js.json()
    print(json.dumps(js.json(),indent=3,ensure_ascii=False))
    return dic

api("xxx.xxx.xxx.xxx")
```

## bgp.he.net爬虫

这个并不像是ip-api.com那样直接返回json，结果是直接由前端解析展示出来，所以我们分析需求应该可以知道，就是请求网页得到HTML，然后解析HTML从中把我们要用到的数据给扣出来，这个应该就是要实现的功能。

https://bgp.he.net/ip/<ip>

https://bgp.he.net/net/<net>

所以我们尽可能的是将对应的文本提取出来就可以

写的简易版本代码，数据未进行排序整理的版本：

```python
#!/usr/bin/python3 
import re
import requests
from bs4 import BeautifulSoup
import urllib.request as urq
#headers = {'User-Agent':'Mozilla/5.0 3578.98 Safari/537.36'}
url="https://bgp.he.net"
headers={
    "Accept": "text/javascript, text/html, application/xml, text/xml, */*",
    "Accept-Encoding": "gzip, deflate, br",
    "Accept-Language": "zh-HK,zh;q=0.9,es;q=0.8,es-ES;q=0.7,zh-TW;q=0.6,zh-CN;q=0.5",
    "Connection": "keep-alive",
    "Cookie": "__utmc=83743493; __utmz=83743493.1583470913.1.1.utmcsr=(direct)|utmccn=(direct)|utmcmd=(none); c=BAgiFDE4MC4xMjUuMTg4LjIzNw%3D%3D--ddd22112e0bac2f0b2d57743ceba49bd0b15b890; _bgp_session=BAh7BzoPc2Vzc2lvbl9pZEkiJTNhNDI0MzE4YzhjOWYyNmQ4NDViNGI2MmZjMzlkMTI1BjoGRUY6EF9jc3JmX3Rva2VuSSIxWVd1SWJ3T001cSttK2NBeDdVWUczekt2WTVyNWNoeVhGVFJYakdaa0tycz0GOwZG--feddfe06b6e24b98127c61f697d0fd0bb08760e4; __utma=83743493.1539647084.1583470913.1583470913.1583474517.2; __utmb=83743493.8.10.1583474517",
    "Host": "bgp.he.net",
    "Origin": "https://bgp.he.net",
    "Referer": "",
    "Sec-Fetch-Dest": "empty",
    "Sec-Fetch-Mode": "cors",
    "Sec-Fetch-Site": "same-origin",
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/80.0.3987.132 Safari/537.36",
    "X-Requested-With": "XMLHttpRequest"
}
href=[]
strs=[]
def prints(lab):
    strss=""
    try:
        for i in lab.children:
            if i.name=="a" and str(i['href'])[0:4]=='/net':
                href.append(url+str(i['href']))
            prints(i)
    except Exception as e:
        if lab=="\n":
            return 0
        if lab.name=="a":
            print(lab)
        strs.append(str(lab))

def spiderip(ip):
    global url
    global headers
    html=""
    urls=url+'/ip/'+ip
    headers["Referer"]=urls
    try:
        r=requests.get(urls,headers=headers)
        html=r.text
    except:
        return {}
    soup=BeautifulSoup(html,"lxml")
    ipinfo=soup.find('div',id="ipinfo")
    tr=ipinfo.find_all('tr')
    for i in tr:
        prints(i)
    whois=soup.find('div',id="whois")
    for i in whois:
        prints(i)
    dns=soup.find('div',id="dns")
    for i in dns:
        prints(i)

def spidernet(urls):
    global headers
    headers["Referer"]=urls
    html=""
    try:
        r=requests.get(url=urls,headers=headers)
        html=r.text
        #print(html)
    except:
        return {}
    soup=BeautifulSoup(html,"lxml")
    netinfo=soup.find('div',id="netinfo")
    #print(netinfo)
    tr=netinfo.find_all('tr')
    print(tr)
    for i in tr:
        prints(i)
    whois=soup.find('div',id="whois")
    for i in whois:
        prints(i)
    dns=soup.find('div',id="dns")
    for i in dns:
        prints(i)



if __name__=="__main__":
    spiderip("120.77.176.157")
    print(href)
    for i in href:
        spidernet(i)
    f=open("./www.txt","ab")
    for i in strs:
        k=i.replace(",","")
        f.write(bytes(k,'UTF-8'))
        f.write(bytes("\n",'UTF-8'))
```

## 参考

https://www.liaoxuefeng.com/wiki/1016959663602400，廖雪峰

https://www.cnblogs.com/lanyinhao/p/9634742.html

https://www.cnblogs.com/zhangxinqi/p/9170312.html

https://www.cnblogs.com/zhangxinqi/p/9218395.html