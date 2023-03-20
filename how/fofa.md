# FOFA

### 如何成为一个合格的“FOFA”工程师

    我看各位师傅都对FOFA打点很感兴趣，我从自己的角度给大家谈谈我打点的几个思路，希望对大家后面的打点能有一点点帮助。

### 0x01：FOFA 的工作原理是什么？

    有些人觉得我不需要关心Fofa的工作原理，我只需要找到我想要的东西就行。其实你可以找到一部分渗透中目标的信息，通过title搜索目标的信息大家都会，你能搜到别人也能搜到，你能打下来别人也肯定能打下来，那么我们该怎么找别人找不到的资产呢？这就需要你去深入理解Fofa的工作机制。

•fofa是什么    fofa的收录是基于端口扫描的收录，底层原理就类似一个扫全网的zmap加上各种指纹识别的脚本和规则来维护的，基于：IP、端口、指纹、部分域名子域名的关系数据库。•什么样的东西可以被fofa收录    上面说过了，fofa是基于**IP**和**端口**来决定什么可以入库，什么东西ta入不了库，也不可能入得了库的。\
**举个栗子**

| 例子                                 | 是否会被收录   | 解释                                        |
| ---------------------------------- | -------- | ----------------------------------------- |
| 111.111.111.111:80                 | 是        | IP、端口两要素                                  |
| 111.111.111.111:80/test/test.html  | 否        | 仅会收录根目录返回结果，二级目录不会收录，fofa不会进行目录扫描         |
| http://baidu.com:80                | 可能       | 某些特定情况下，可能会扫到baidu.com对应的IP，并且该IP未作针对性防护  |
| http://baidu.com:80/test/test.html | 可能（概率很小） | 唯一一种可能的情况就是/test/目录单独解析到某个IP，或服务由某个IP节点提供 |

    上面列举了一些可能被收录的和不会被收录的资产，fofa搜索开始之前就应该了解，自己搜索什么可以搜到东西，而搜索什么不可能找到东西。

### 0x02：Title & Header & Body & Html 的关系？

#### - Title

    Title顾名思义就是网站的标题，那么我们仔细观察一下常见的标题构成要素。根据标题构成自己手动分词！\


**接着举栗**

```
首页 - XX市XX医院
XX市XX医院 - 首页
XX市XX医院
XX市附属XX医院
XX医院
AA医院
AA市XX医院
XX市XX医院与AA医院合作系统
```

&#x20;      分析上面title构成编写一个搜索语法，查找尽可能多的**XX市XX医院**资产

```
title="XX市XX医院"
```

&#x20;      这一条可以匹配的是标题中含有“XX市XX医”的资产，而标题为"XX医院"的某些资产也可能是XX市XX医院的资产,我们就无法获取这部分资产信息。\


```
title="XX市" && title="XX医院"
```

&#x20;      这条规则比上面 的规则可以匹配到更多的资产，但是会搜索到目标之外的结果”XX市XX医院与AA医院合作系统“所以搜索的时候**拆分title**来搜索可以获得更多的搜索结果和更大的搜索范围，但是可能会带来脏数据，需要我们人工辨别是否为目标所属资产。

注：了解：&&、||、==、!=都是什么意思。

#### - Header/Body

    需要注意的点和Title类似，不做过多解释。

#### - Html（source code）

    当我们分别查找title、header、body的关键字的时候总觉得太麻烦，能不能直接搜索一次就包含所有的搜索结果呢？答案是肯定的。\
**再次举栗**

```markup
<html>
<head>
<title>首页 - XX管理系统</title>
</head>
<body>
<p>body welcome to my web</p>
<p>user：</p>
<p>pass：</p>
</body>
<footer>版权信息：XX市XX医院<footer>
</html>
```

&#x20;     上面的代码可以看到，footer里面包含我们的目标关键字，但是fofa中并没有专门搜索footer的语法，那么我们怎么把这个结果搜索到呢？\
如下代码就可以了：

```
"XX市XX医院"
```

&#x20;      上面这个代码虽然官方没有解释，但是测试是可以搜索==整个source code==里面的关键字的。\
&#x20;      需要注意的是，这个语法会搜到==很多辣鸡数据==，有多多呢？根据网站类型来说，很多仿冒的网站、新闻网站、色情网站都会插入大量辣鸡数据增加搜索引擎收录的概率，这也会影响我们搜索结果的筛选。有没有办法过滤掉这些辣鸡数据呢？有办法！

```
"XX市XX医院" && country="CN" && region!="HK"
```

&#x20;      看一眼这个语法，应该不用解释了，过滤掉容易出现辣鸡网站的地区==只要中国的结果==，其中==容易出现辣鸡数据的HK==也进行过滤，会清爽很多。\
&#x20;      其他一些语法的组合自己尝试吧：

```
title="登陆" && "XX市" && "XX医院" && country="CN" && region!="HK"
title="管理" && "XX市" && "XX医院" && country="CN" && region!="HK"
title="系统" && "XX市" && "XX医院" && country="CN" && region!="HK"
title="测试" && "XX市" && "XX医院" && country="CN" && region!="HK"
等等···
```

**- 某些特殊关注点**

```markup
<html>
<head>
<title>首页 - XX管理系统</title>
<a href="/example/html/cnm.html">
</head>
<body>
<p>body welcome to my web</p>
<p>user：</p>
<p>pass：</p>
</body>
<footer>版权信息：XX市XX医院<footer>
<footer>hackersb<footer>
</html>
```

    比如某些网站会出现一些独特的代码片段，正常网站基本上不会出现的==特征点==同样可以作为我们的搜索语法来使用：

```
"/example/html/cnm.html" && country="CN" && region!="HK"
"<footer>hackersb<footer>" && country="CN" && region!="HK"
```

### 0x03：某些傻X API接口 的特殊姿势？

    有些API接口，直接访问会报错：\
**举栗举栗**

```markup
<!DOCTYPE html>
<html>
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>MiniGo API</title>
    <style type="text/css">
        html, body {font-size: 1em; font-family: Arial, Helvetica, sans-serif; padding: 0;margin: 0;}
        a, a:hover, a:visited {color: blue;}
        ul li {font-size: 14px;}
        code {font-size: 14px; font-family: Helvetica, Arial, sans-serif; color:blue; margin: 0 5px; }
        div.header { height: 30px; line-height: 30px; background: #159C94;color:white; font-weight: bold; padding:0 0 0 48px;}
        div.header img.logo {position: absolute; left: 10px; top: 3px;}
        div.header img.hisense {position: absolute; right: 10px; top: 6px;}
        div.body {padding: 10px;}
        div.alert {vertical-align:middle; border:2px solid red;background: yellow url('/img/alert.png') no-repeat 0 4px; padding: 12px 12px 12px 72px;color:red;min-height: 36px;}
        div.alert p {margin: 0;}
        div.footer {position: absolute; height: 60px; bottom: 0;left: 0;}
    </style>
</head>
<body>
    <div class="header"><img class="logo" src="/img/logo.png" width="24" height="24" /> MINIGO 微信小程序<img class="hisense" src="/img/hisense.png"/></div>
    <div class="body">

欢迎使用XX WebAPI中间服务，当前版本为V2.0.1.5
    </div>
    <div class="footer"></div>
</body>
</html>
```

以上这种情况，我反手就是一个：

```
"欢迎使用XX WebAPI中间服务，当前版本为V2.0.1.5"
"欢迎使用XX WebAPI中间服务"
```

    全部都出来了，各个版本肯定也出来了，肯定都是他家的！（分清自研和商用系统）\
    ==你问我，你又日不下这API，搜出来有何屌用？==\
    是哦，我日不下来，我有大哥们呢！\
    其实还是有用的，我们可以得到以下有用的信息：

1. IP资产
2. 关联相关IP段
3. 端口扫描

    这种情况紧跟的就是：

```
ip="111.111.111.111/24"
```

    ==这回可以找你能日下的系统了吧！==（注意识别资产归属）

### 0x04：某些傻X 证书 的特殊姿势？

&#x20;   下面这图肯定都见过，但是你认真研究过这个页面吗？

![](<../.gitbook/assets/image (774).png>)

点开高级进行分析：

&#x20; 看到证书来自baidu.com了吧，那这资产是不是百度的？这傻吊也是可以用FOFA来搜索的：

*

```
cert="baidu.com"
```

**那么我们来详细观察一个证书的构成：**

查看证书实例！

```markup
HTTP/1.1 403 Forbidden
Server: JSP3/2.0.14
Date: Mon, 19 Oct 2020 06:07:15 GMT
Content-Type: text/html
Content-Length: 152
Connection: close
Certificate

Version: v3
Serial Number: 35388244279832734960132917320
Signature Algorithm: SHA256-RSA

Issuer:
Country: BE
Organization: GlobalSign nv-sa
CommonName: GlobalSign Organization Validation CA - SHA256 - G2

Validity:
Not Before: 2020-04-02 07:04 UTC
Not After : 2021-07-26 05:31 UTC

Subject:
Country: CN
Province: beijing
Locality: beijing
Organizational Unit: service operation department
Organization: Beijing Baidu Netcom Science Technology Co., Ltd
CommonName: baidu.com

Subject Public Key Info:
Public Key Algorithm: RSA
Public Key:
Exponent: 65537
Public Key Modulus: (2048 bits) :
C1:A9:B0:AE:47:1A:D2:57:EB:1D:15:1F:6E:5C:B2:E4:
F8:0B:20:DB:EA:00:DF:29:FF:A4:6B:89:26:4B:9F:23:
2F:EC:57:B0:8A:B8:46:40:2A:7E:BC:DC:5A:45:97:4F:
AD:41:0E:BC:20:86:4B:0C:5D:55:21:47:E2:31:3C:57:
A7:EC:99:47:EB:47:0D:72:D7:C8:16:54:75:EF:D3:45:
11:0F:4B:CE:60:7A:46:5C:28:74:AE:8E:1B:BE:D8:70:
66:7B:A8:93:49:28:D2:A3:76:94:55:DE:7C:27:F2:0F:
F7:98:0C:AD:86:DA:C6:AE:FD:9F:F0:D9:81:32:9A:97:
E3:21:EE:04:92:96:E4:78:11:E5:C4:10:0E:10:31:7A:
4A:97:A0:EB:C7:9B:C4:DA:89:37:A9:C3:37:D7:56:B1:
7F:52:C7:D9:26:0A:D6:AF:38:16:B1:6D:FB:73:79:B1:
68:79:03:90:EB:88:7B:8C:48:91:98:51:A5:07:94:86:
A5:78:46:79:8F:58:9B:E9:35:59:A7:F1:7B:57:31:0A:
90:CF:24:CE:0D:24:E7:92:B2:6A:E9:E6:96:37:0A:B8:
7C:87:2F:74:D2:5C:E8:4B:0A:5F:66:18:A7:41:86:CF:
26:A6:08:8E:A5:49:17:92:53:B3:91:A5:CF:53:B0:31

Authority Key Identifier:
96:DE:61:F1:BD:1C:16:29:53:1C:C0:CC:7D:3B:83:00:40:E6:1A:7C

Subject Key Identifier:
9E:C9:79:D7:E9:5B:AB:8A:16:CC:32:8E:C6:99:E6:9F:20:42:35:87

CRL Distribution Points:http://crl.globalsign.com/gs/gsorganizationvalsha2g2.crl

Basic Constraints:
CA : false
Path Length Constraint: UNLIMITED

OCSP Server:http://ocsp2.globalsign.com/gsorganizationvalsha2g2

Issuing Certificate URL:http://secure.globalsign.com/cacert/gsorganizationvalsha2g2r1.crt

Key Usage:
Digital Signature
Key Encipherment

Extended Key Usage:
Server Auth
Client Auth

DNS Names:baidu.combaifubao.comwww.baidu.cnwww.baidu.com.cnmct.y.nuomi.com
apollo.autodwz.cn
*.baidu.com
*.baifubao.com
*.baidustatic.com
*.bdstatic.com
*.bdimg.com
*.hao123.com
*.nuomi.com
*.chuanke.com
*.trustgo.com
*.bce.baidu.com
*.eyun.baidu.com
*.map.baidu.com
*.mbd.baidu.com
*.fanyi.baidu.com
*.baidubce.com
*.mipcdn.com
*.news.baidu.com
*.baidupcs.com
*.aipage.com
*.aipage.cn
*.bcehost.com
*.safe.baidu.com
*.im.baidu.com
*.baiducontent.com
*.dlnel.com
*.dlnel.org
*.dueros.baidu.com
*.su.baidu.com
*.91.com
*.hao123.baidu.com
*.apollo.auto
*.xueshu.baidu.com
*.bj.baidubce.com
*.gz.baidubce.com
*.smartapps.cn
*.bdtjrcv.com
*.hao222.com
*.haokan.com
*.pae.baidu.com
*.vd.bdstatic.comclick.hm.baidu.comlog.hm.baidu.comcm.pos.baidu.comwn.pos.baidu.comupdate.pan.baidu.com

Certificate Signature Algorithm: SHA256-RSA
Certificate Signature:
BC:DC:02:D0:D9:DE:8C:C5:E2:D9:FE:4D:EF:BA:D1:22:
8B:34:42:59:84:92:31:82:D5:0A:BC:40:35:DB:06:B2:
13:6E:C8:CF:01:F1:5F:C0:E7:B7:34:37:3A:A8:08:F2:
9F:32:D5:F9:20:80:9F:BF:D3:FF:6D:47:9C:76:D1:CB:
F1:C7:F1:DB:83:33:37:E5:3F:18:A7:00:E2:BD:DA:FE:
4F:29:45:57:87:78:5F:53:85:0D:B3:A3:5C:63:93:FE:
E0:26:5E:F9:92:8C:ED:76:A3:5F:39:E6:22:05:36:C5:
32:73:D0:CD:51:AA:C8:C3:1F:A8:AC:5B:26:B7:D9:94:
60:08:81:81:D3:F5:B7:7A:4F:DF:39:21:58:33:B5:15:
63:02:8C:B8:22:EA:D9:7A:74:EC:5A:41:BB:3D:A7:C9:
E2:40:21:EA:34:1A:4A:ED:73:60:46:C7:96:3B:99:E4:
F5:E5:92:13:CE:F4:3C:16:D5:62:0F:BA:0E:99:AE:5C:
A5:2D:34:D8:9A:55:B7:58:44:CE:01:38:BB:D0:76:2C:
64:DE:8D:00:2B:99:E2:DD:61:10:ED:C0:B0:5E:E5:AA:
37:40:D8:7C:13:37:5D:05:5F:61:EE:69:4B:DF:E4:EC:
CF:F8:F2:AE:A5:5F:55:2B:0F:31:F2:64:0A:53:AB:EB
```

&#x20;      通过以上结果可以得出不光【baidu.com】，属于百度的好多域名都知道了，知道别人日站为什么不用开工具扫什么子域名扫IP了吧，IP、域名都收集齐了，根本不会出错！

### 0x05：某些傻X CDN 的特殊姿势？

提供几个思路吧，都大同小异：

1.通过证书查询方式

```
cert="baidu.com"
```

这个可以绕过一部分SSL网站的CDN直接看到真实IP

1.通过某些主页特殊关键字

```
"特殊关键字"
```

某些网站没有对IP直接访问做限制，可以直接访问

1.某些特殊接口 比如你访问某个接口给你返回一个==baidu OK！==

```
"baidu OK！"
```

等等
