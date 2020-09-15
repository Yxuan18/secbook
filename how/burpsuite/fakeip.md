# FAKE-IP

### 1、下载地址

https://github.com/TheKingOfDuck/burpFakeIP

### 2、安装插件

1、解压到本地

2、打开burpsuite，选择Extender，Add

![](../../.gitbook/assets/image%20%28341%29.png)

3、选择下载好python插件，选择下一步

![](../../.gitbook/assets/image%20%28297%29.png)

4、安装成功

![](../../.gitbook/assets/image%20%28369%29.png)

### 3、使用方法

1、伪造指定ip，右击抓到的数据包，选择fakeip，inputIP

![](../../.gitbook/assets/image%20%28300%29.png)

2、输入想要用的ip地址，点击确定，自动添加

![](../../.gitbook/assets/image%20%28311%29.png)

3、伪造本地ip，右击数据包选择127.0.0.1，自动生成如下数据包

![](../../.gitbook/assets/image%20%28338%29.png)

```text
GET / HTTP/1.1
Host: 192.168.1.135:8002
User-Agent: Mozilla/5.0 (Windows NT 6.1; WOW64; rv:52.0) Gecko/20100101 Firefox/52.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Cookie: ASPSESSIONIDAACSRSQS=CJLFMKFBLOABHEPKPBEKLPOA; ASPSESSIONIDAACSRTRT=HOAFJLFBABBHLECCHJEGMIMO
DNT: 1
Connection: closeUpgrade-Insecure-Requests: 1
X-Forwarded-For:127.0.0.1
X-Forwarded:127.0.0.1
Forwarded-For:127.0.0.1
Forwarded:127.0.0.1
X-Forwarded-Host:127.0.0.1
X-remote-IP:127.0.0.1
X-remote-addr:127.0.0.1
True-Client-IP:127.0.0.1
X-Client-IP:127.0.0.1
Client-IP:127.0.0.1
X-Real-IP:127.0.0.1
Ali-CDN-Real-IP:127.0.0.1
Cdn-Src-Ip:127.0.0.1
Cdn-Real-Ip:127.0.0.1
CF-Connecting-IP:127.0.0.1
X-Cluster-Client-IP:127.0.0.1
WL-Proxy-Client-IP:127.0.0.1
Proxy-Client-IP:127.0.0.1
Fastly-Client-Ip:127.0.0.1
True-Client-Ip:127.0.0.1
```

3、伪造随机IP，右击数据包选择randomIP，生成如下数据包

```text
GET / HTTP/1.1
Host: 192.168.1.135:8002
User-Agent: Mozilla/5.0 (Windows NT 6.1; WOW64; rv:52.0) Gecko/20100101 Firefox/52.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Cookie: ASPSESSIONIDAACSRSQS=CJLFMKFBLOABHEPKPBEKLPOA; ASPSESSIONIDAACSRTRT=HOAFJLFBABBHLECCHJEGMIMO
DNT: 1
Connection: closeUpgrade-Insecure-Requests: 1
X-Forwarded-For:37.120.247.234
X-Forwarded:37.120.247.234
Forwarded-For:37.120.247.234
Forwarded:37.120.247.234
X-Forwarded-Host:37.120.247.234
X-remote-IP:37.120.247.234
X-remote-addr:37.120.247.234
True-Client-IP:37.120.247.234
X-Client-IP:37.120.247.234
Client-IP:37.120.247.234
X-Real-IP:37.120.247.234
Ali-CDN-Real-IP:37.120.247.234
Cdn-Src-Ip:37.120.247.234
Cdn-Real-Ip:37.120.247.234
CF-Connecting-IP:37.120.247.234
X-Cluster-Client-IP:37.120.247.234
WL-Proxy-Client-IP:37.120.247.234
Proxy-Client-IP:37.120.247.234
Fastly-Client-Ip:37.120.247.234
True-Client-Ip:37.120.247.234
```

4、随机ip爆破，伪造随机ip爆破是本插件最核心的功能。

![](../../.gitbook/assets/image%20%28345%29.png)

将数据包发送到Intruder模块,在Positions中切换Attack type为Pitchfork模式,选择好有效的伪造字段,以及需要爆破的字段

![](../../.gitbook/assets/image%20%28299%29.png)

将Payload来源设置为Extensin-generated,并设置负载伪fakeIpPayloads,然后设置第二个变量。

![](../../.gitbook/assets/image%20%28322%29.png)

点击Start attack开始爆破.



