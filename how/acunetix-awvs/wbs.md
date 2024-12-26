# 编写AWVS脚本探测web services

### 0x01思路 <a href="#id-0x01-si-lu" id="id-0x01-si-lu"></a>

我们让AWVS爬取到的每一个路径都添加/services，然后去访问这个构造好的路径。如果存在该页面，则分析返回结果中是否存在“wdsl”字符，若存在则说明该站点存在web services服务。

### 0x02编写代码 <a href="#id-0x02-bian-xie-dai-ma" id="id-0x02-bian-xie-dai-ma"></a>

#### 新建报告模板 <a href="#xin-jian-bao-gao-mo-ban" id="xin-jian-bao-gao-mo-ban"></a>

AWVS》Tools》Vulnerability Editor

图1-新建报告模板

![图1-新建报告模板](https://gv7.me/articles/2017/Writing-AWVS-scripts-to-detect-Web-Services/add_xml.png)

填写好漏洞相关信息

图2-填写漏洞信息

![图2-填写漏洞信息](https://gv7.me/articles/2017/Writing-AWVS-scripts-to-detect-Web-Services/2.png)

#### 新建探测脚本 <a href="#xin-jian-tan-ce-jiao-ben" id="xin-jian-tan-ce-jiao-ben"></a>

找到AWVS的/data/script/folder目录

图3-新建探测脚本

![图3-新建探测脚本](https://gv7.me/articles/2017/Writing-AWVS-scripts-to-detect-Web-Services/3.png)

由于我们需要AWVS在爬取到目录时就检查一次该目录是否存在web services。所以我们需要在该目录下的PerFolder文件夹，新建一个名为Web\_Services.script的脚本文件。代码如下：

```java
var target = new THTTPJob(); //实例化一个HTTP任务
var dir = getCurrentDirectory();//获取当前路径
target.url = new TURL(scanURL.url+ dir.fullPath + "/services");//构造请求url
target.execute();//执行http请求

var wsRes = target.response.body;//获取http请求内容
if(!target.wasError && !target.notFound ){//判断是否访问错误或者是404
	if(wsRes.indexOf('wsdl') != -1){
		logWarning(scanURL.url+dir.fullPath+'----->this web services is exists!!!');//在日志栏显示该调式信息
		var ri = new TReportItem();//新建一个报告结果，返回给扫描器界面
		ri.loadFromFile('Web_Services.xml');//载入模板
		ri.severity = "high"//影响等级
		ri.affects = dir.fullPath + "/services";
		ri.Request = target.Request.headersString;//测试请求HTTP头输出到界面
		ri.response = target.response.body;//测试请求HTTP响应内容输出到界面
		ri.fullResponse = target.fullResponse;//测试请求的完整HTTP响应内容输出到界面
		
		//ri.description = "web services";
		ri.addReference("how do sql inject web services","http://gv7.me/2017/08/12/how-do-sql-inject-web-services/");
		
		AddReportItem(ri);
	}
	else
	{
		logError(scanURL.url+dir.fullPath+"----->This's not web services!!!");
	}
}else{
		logWarning(scanURL.url+dir.fullPath+"notFound web services!!!!");
}
```

图4-代码

![图4-代码](https://gv7.me/articles/2017/Writing-AWVS-scripts-to-detect-Web-Services/4.png)

### 0x3测试 <a href="#id-0x3-ce-shi" id="id-0x3-ce-shi"></a>

去网上随便找一个测试站点，该站点/pptx/路径下存在web services的wsdl列表。

图5-测试站点

![图5-测试站点](https://gv7.me/articles/2017/Writing-AWVS-scripts-to-detect-Web-Services/5.png)

为了方便测试，我们新建一个test策略，策略包含我们写的脚本。

图6-新建test策略

![图6-新建test策略](https://gv7.me/articles/2017/Writing-AWVS-scripts-to-detect-Web-Services/6.png)

扫描选择我们的新建的策略

图7-测试扫描设置

![图7-测试扫描设置](https://gv7.me/articles/2017/Writing-AWVS-scripts-to-detect-Web-Services/7.png)

扫描结果中发现，已经找到web services\
图8-测试扫描结果

![图8-测试扫描结果](https://gv7.me/articles/2017/Writing-AWVS-scripts-to-detect-Web-Services/8.png)

后面我在这个代码基础上泄露发现SVN泄露的脚本，大家想让AWVS发现更多漏洞，可以自己尝试去写写。

### 0x4AWVS脚本编写资料 <a href="#id-0x4awvs-jiao-ben-bian-xie-zi-liao" id="id-0x4awvs-jiao-ben-bian-xie-zi-liao"></a>

如果想了解更多编写脚本的资料。可以下载以下推荐的资料，若你有更好，欢淫共享感激不尽。

#### 官方SDK文档 <a href="#guan-fang-sdk-wen-dang" id="guan-fang-sdk-wen-dang"></a>

```
https://www.acunetix.com/resources/sdk/
```

#### 官方开发工具包 <a href="#guan-fang-kai-fa-gong-ju-bao" id="guan-fang-kai-fa-gong-ju-bao"></a>

工具包里有一个文档，和3个脚本例子。大家可以参考一下。

```
http://www.acunetix.com/download/tools/WVSSDK.zip
```

#### 解密的扫描脚本 <a href="#jie-mi-de-sao-miao-jiao-ben" id="jie-mi-de-sao-miao-jiao-ben"></a>

大家可能发现在/data/script/文件夹下的所有脚本都加密了，我们无法查看源码。这对于我们想学习编写脚本的童鞋很是不利。这里给大家发个福利：〇〇木一大神解密AWVS 10.5的script文件夹下所有脚本。

```
https://github.com/c0ny1/awvs_script_decode
```

百度云下载(base64)

```
dXJs77yaaHR0cHM6Ly9wYW4uYmFpZHUuY29tL3MvMXNscjRIUHogcHdk77yaYjNtbw==
```
