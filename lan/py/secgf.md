# 安全编码规范

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

```text
def xss_test(request):    
name = request.GET['name']    
return HttpResponse('hello %s' %(name))
```

在代码中一搜，发现有大量地方使用，比较正确的使用方式如下：  


更好的就是对输入做限制，比如说一个正则范围，输出使用正确的api或者做好过滤。

## 3 CSRF <a id="3-CSRF"></a>

对系统中一些重要的操作要做CSRF防护，比如登录，关机，扫描等。django 提供CSRF中间件`django.middleware.csrf.CsrfViewMiddleware`,写入到settings.py的中间件即可。  


## 4 命令注入 <a id="4-&#x547D;&#x4EE4;&#x6CE8;&#x5165;"></a>

审计代码过程中发现了一些编写代码的不好的习惯，体现最严重的就是在命令注入方面，本来python自身的一些函数库就能完成的功能，偏偏要调用os.system来通过shell 命令执行来完成，老实说最烦这种写代码的啦。下面举个简单的例子：  


很显然这段代码是存在问题的，因为fullname是用户可控的。正确的做法是不使用os.system接口，改成python自有的库函数，这样就能避免命令注入。python的三种删除文件方式：  
（1）shutil.rmtree 删除一个文件夹及所有文件  
（2）os.rmdir 删除一个空目录  
（3）os.remove，unlink 删除一个文件

使用了上述接口之后还得注意不能穿越目录，不然整个系统都有可能被删除了。常见的存在命令执行风险的函数如下：  


推荐使用subprocess模块，同时确保shell=True未设置，否则也是存在注入风险的。

## 5 sql注入 <a id="5-sql&#x6CE8;&#x5165;"></a>

如果是使用django的api去操作数据库就应该不会有sql注入了，但是因为一些其他原因使用了拼接sql，就会有sql注入风险。下面贴一个有注入风险的例子：  


这一段代码就是就是因为eval的参数不可控，导致任意代码执行，正确的做法就是literal.eval接口。再取个pickle.loads的例子：  


## 7 文件操作 <a id="7-&#x6587;&#x4EF6;&#x64CD;&#x4F5C;"></a>

文件操作主要包含任意文件下载，删除，写入，覆盖等，如果能达到写入的目的时基本上就能写一个webshell了。下面举个任意文件下载的例子：  


这段代码就存在着任意.lic文件下载的问题，没有做好限制目录穿越，同理

## 8 文件上传 <a id="8-&#x6587;&#x4EF6;&#x4E0A;&#x4F20;"></a>

### 8.1 任意文件上传 <a id="8-1-&#x4EFB;&#x610F;&#x6587;&#x4EF6;&#x4E0A;&#x4F20;"></a>

这里主要是未限制文件大小，可能导致ddos，未限制文件后缀，导致任意文件上传，未给文件重命名，可能导致目录穿越，文件覆盖等问题。

### 8.2 xml，excel等上传 <a id="8-2-xml&#xFF0C;excel&#x7B49;&#x4E0A;&#x4F20;"></a>

在我们的产品中经常用到xml来保存一些配置文件，同时也支持xml文件的导出导入，这样在libxml2.9以下就可能导致xxe漏洞。就拿lxml来说吧：  


这是因为在lxml中默认采用的XMLParser导致的：  


关注其中两个关键参数，其中resolve\_entities=True,no\_network=True,其中resolve\_entities=True会导致解析实体，no\_network会为True就导致了该利用条件比较有效，会导致一些ssrf问题，不能将数据带出。在python中xml.dom.minidom,xml.etree.ElementTree不受影响

## 9 不安全的封装 <a id="9-&#x4E0D;&#x5B89;&#x5168;&#x7684;&#x5C01;&#x88C5;"></a>

### 9.1 eval 封装不彻底 <a id="9-1-eval-&#x5C01;&#x88C5;&#x4E0D;&#x5F7B;&#x5E95;"></a>

仅仅是将`__builtings__`置为空，如下方式即可绕过,可参见[bug84179](http://xxlegend.com/2015/07/30/Python%E5%AE%89%E5%85%A8%E7%BC%96%E7%A0%81%E5%92%8C%E4%BB%A3%E7%A0%81%E5%AE%A1%E8%AE%A1/)

```text
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

## 10 总结 <a id="10-&#x603B;&#x7ED3;"></a>

一切输入都是不可靠的，做好严格过滤。
