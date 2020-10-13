# 漏洞挖掘

## 漏洞挖掘步骤： <a id="&#x6F0F;&#x6D1E;&#x6316;&#x6398;&#x6B65;&#x9AA4;&#xFF1A;"></a>

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

## OWASP Top10 <a id="owasp-top10"></a>

> **1.注入：**SQL注入、OS注入\(命令执行\)、LDAP注入 **2.失效的身份认证和会话管理：**弱口令爆破、不安全的散列密码加密\(MD5爆破\) **3.敏感数据泄漏：**源码泄漏、配置文件暴露、www.zip备份文件、默认后台 **4.XML外部实体\(XXE\)** **5.失效的访问控制：**管理页面仅能管理员权限访问；越权漏洞\(垂直越权、水平越权\); JWT-Cookie伪造 **6.安全配置错误：**开放了不必要的功能\(445端口、网页-默认安装页面未删除、页面报错\)、默认密码或空密码 **7.跨站脚本\(XSS\)** **8.不安全的反序列化：**java、php、python **9.使用含有已知漏洞的组件：**未打补丁的系统和组件、使用有已知漏洞的框架版本 **10.不足的日志记录和监控：**代码被删除，日志被修改，无法溯源；应该记录登陆失败次数；监控问题没被管理员响应

## 渗透测试CheckList <a id="&#x6E17;&#x900F;&#x6D4B;&#x8BD5;checklist"></a>

![](https://wiki.wgpsec.org/images/image-20200802124213681.png)

![](https://wiki.wgpsec.org/images/image-20200802125146066.png)

![](https://wiki.wgpsec.org/images/image-20200802130517963.png)

> 这里并没有列全，反正就是各种Day去打就是了

## 漏洞扫描 <a id="&#x6F0F;&#x6D1E;&#x626B;&#x63CF;"></a>

**【AWVS爬虫 + Xray被动扫描】联动**

```text
1、xray开启监听
./xray webscan --listen 0.0.0.0:1111 --html-output resualt.html

2、AWVS添加任务，走xray代理,选择Crawl Only即仅使用爬虫
```

## 任意文件读取（下载） <a id="&#x4EFB;&#x610F;&#x6587;&#x4EF6;&#x8BFB;&#x53D6;&#xFF08;&#x4E0B;&#x8F7D;&#xFF09;"></a>

**JAVA站文件读取漏洞，下载网站源码工具**

 [https://github.com/LandGrey/ClassHound](https://github.com/LandGrey/ClassHound)

 [https://github.com/Artemis1029/Java\_xmlhack](https://github.com/Artemis1029/Java_xmlhack)

## 社工打点 <a id="&#x793E;&#x5DE5;&#x6253;&#x70B9;"></a>

邮件钓鱼

伪装求职者欺骗HR（传word或PDF马）

伪装客户欺骗销售（传木马）

伪装客户联系技术支持（获取系统密码）

