# 流量特征修改

### **0x01 前言** <a id="h2-1"></a>

nmap是渗透中尝尝用到的工具之一，在信息收集阶段经常用到，现在主流的流量分析设备也将其流量加入了特征库，

为了防止在探测阶段IP就被封掉，对其的流量特征做一些简单的修改有点用的。

由于没有厂商设备检测，故以下只是学习记录一下思路。具体效果还待验证。

**参考链接**

如何修改nmap， 重新编译，bypass emergingthreats 的公开ids规则：

https://xz.aliyun.com/t/6002

nmap端口扫描技术：

https://nmap.org/man/zh/man-port-scanning-techniques.html

NmapbypassIDS：

https://github.com/al0ne/Nmap\_Bypass\_IDS

### **0x02 环境** <a id="h2-2"></a>

VM虚拟机：

192.168.1.113 开放135，3389 防火墙禁止445入站

![](../../.gitbook/assets/image%20%28529%29.png)

ubuntu：

编译安装nmap用，kali下编译安装存在点问题，坑太多了，以后有时间再去踩。

### **0x03 nmap探测的常用几种方式** <a id="h2-3"></a>

#### **-sS \(SYN扫描\)** <a id="h3-1"></a>

nmap默认端口扫描方式，执行半开扫描，不完成TCP握手流程。只向目标端口发送一个SYN报文，然后等待响应。

SYN/ACK表示端口在监听 \(开放\)，而 RST \(复位\)表示没有监听者。

如果多次重发后仍没响应， 该端口就被标记为被过滤。

使用抓包工具可以完整的看到探测过程。

![](../../.gitbook/assets/image%20%28528%29.png)

#### **-sT \(TCP扫描\)** <a id="h3-2"></a>

一般不推荐使用，因为会留下连接日志。

另外在调用一些高级扫描时（如HTTP），会调用这种连接方式。

使用抓包工具看其探测过程

![1600245394\_5f61ce927c86548174248.png!small](https://image.3001.net/images/20200916/1600245394_5f61ce927c86548174248.png!small)

#### **-sU（UDP扫描）** <a id="h3-3"></a>

DNS，SNMP，和DHCP是常常开放UDP的几个服务，有些情况下会使用到。

由于UDP是无连接的，所以会面临响应探测问题，探测时的端口是否开放还是被过滤的判断，会让Nmap超时然后再探测，以防探测帧或者响应丢失，导致探测时间增长。

关闭的端口常常是更大的问题。它们一般发回一个ICMP端口无法到达错误。

但是不像TCP发送的RST报文，许多主机在默认情况下限制ICMP端口不可到达消息。

如：Linux 2.4.20内核限制一秒钟只发送一条目标不可到达消息。

抓包看一下，当只看到两个UDP无内容包时，懵了一下。

查了一下发现除了某些特定端口会有响应返回，如137 用的NBNS，其他的全部都是没有返回，原因是因为这台机器禁PING了，就是ICMP的返回包过不来。

所以没法判断端口是否关闭。

![1600245422\_5f61ceaeb6398ed33981f.png!small](https://image.3001.net/images/20200916/1600245422_5f61ceaeb6398ed33981f.png!small)

修改防火墙设置。

允许文件和打印机共享后确实可以ping主机了，但是ICMP回包还是有问题。

后来索性把防火墙关掉。

![](https://image.3001.net/images/20200916/1600245433_5f61ceb932d45c98a5e31.png!small)

就可以明显看到其是通过返回包来进行判断的。

![1600245445\_5f61cec51dbafd4bd2fcf.png!small](https://image.3001.net/images/20200916/1600245445_5f61cec51dbafd4bd2fcf.png!small)

#### **-sN；-sF；-sX \(TCP Null，FIN，and Xmas扫描\)** <a id="h3-4"></a>

这个还是挺有意思的，首先这个不适用扫描windows、Cisco、bsdi、IBM的一些服务器，因为并不是完全遵守RFC 793这个协议。

这个协议会存在这种情况，当端口关闭时，任何不包含SYN，RST，或者ACK位的报文会导致 一个RST返回，而当端口开放时，应该没有任何响应。

所以只要不包含SYN，RST，或者ACK， 任何其它三种\(FIN，PSH，and URG\)的组合都行。

而刚刚上面说的那些他们并不遵守这个，他们不管端口开放或关闭，都返回一个RST，导致Nmap判断错误。

-sN 不设置任何标志位

-sF 只设置FIN标志位

-sX 设置FIN，PSH，和URG标志位

看一下探测过程，如果没加参数，默认会先发送ICMP请求。

![640?wx\_fmt=png](https://image.3001.net/images/20200917/1600334326_5f6329f618d5368487a06.png!small)

#### **-sA（ACK 扫描）** <a id="h3-5"></a>

用于探测防火墙状态。ACK扫描探测报文只设置ACK标志位。

当扫描未被过滤的系统时， `open`\(开放的\)和 `closed`\(关闭的\) 端口 都会返回RST报文。

Nmap把它们标记为 `unfiltered`\(未被过滤的\)，无返回或者返回ICMP错误响应时标记为filtered。

防火墙关闭状态下。返回**unfiltered**

![](https://image.3001.net/images/20200916/1600245515_5f61cf0b25adc18669dd2.png!small)

防火墙开启状态下。返回**filtered**

![1600245525\_5f61cf157a5c7d721ff7e.png!small](https://image.3001.net/images/20200916/1600245525_5f61cf157a5c7d721ff7e.png!small)

#### **-scanflags （自定义扫描）** <a id="h3-6"></a>

可以使用 `URG`， `ACK`， `PSH`， `RST`， `SYN`，and `FIN`的任何组合，进行发包。详细可以自己组合定制

#### **-sI（Idlescan）** <a id="h3-7"></a>

高级隐藏扫描。利用僵尸网络执行扫描。详细可看文章

（文章链接：https://nmap.org/book/idlescan.html）

#### **-sV （版本检测）-O （系统检测）** <a id="h3-8"></a>

可以看到在探测的时候会有标志和固定长度字符串问题。

IDS识别nmap扫描一般都是根据UDP data区域填充的'C'字符串,ICMP填充的是0（正常windows下是a-z，Linux下是0-9。

莫慌，接下就学习一下怎么去改这些个文件。

![](https://image.3001.net/images/20200916/1600245538_5f61cf225b37a917f6921.png!small)

![1600245549\_5f61cf2dc7f7233ae62f2.png!small](https://image.3001.net/images/20200916/1600245549_5f61cf2dc7f7233ae62f2.png!small)

### **0x04 nmap的流量特征修改** <a id="h2-4"></a>

#### **Win值修改** <a id="h3-9"></a>

通过观察可以发现nmap在使用SYN扫描时Windows的窗口值值固定是1024。

（PS ：window 关键字用于检查特定的TCP窗口大小）

![](https://image.3001.net/images/20200916/1600245563_5f61cf3bc2bd6bfc505e2.png!small)

下面是正常连接3389时，发送的数据包。可以看到win值明显不一样。

![](https://image.3001.net/images/20200916/1600245573_5f61cf450ca60945a48f2.png!small)

修改**tcpip.cc**文件中tcp-&gt;th\_win的值，查询TCP中win这个值的信息发现，

默认最大为65535。

所以应该在此范围内都可以。

但是要考虑已公开的规则，如之前大佬写的bypass emergingthreats这篇，这个就过滤了2048 1024 3072 4096。

![](https://image.3001.net/images/20200916/1600245582_5f61cf4ee1cae0d632746.png!small)

后来因某些原因，把nmap编译到了云服务器上，抓包的话就需要tcpdump了。

```text
tcpdump -i eth0 -t -s 0 -c 100 host IP
```

![](https://image.3001.net/images/20200916/1600245592_5f61cf58e625270f9cd5a.png!small)

#### **关键词修改** <a id="h3-10"></a>

根据规则，一个一个去修改文件即可。nmap，nm，nm@p，OR sqlspider等等，

主要的就是SIP文件和一些常用的脚本文件。

这些个就是从emergingthreats的规则中提取的。

![](https://image.3001.net/images/20200916/1600245616_5f61cf707e09147fbe263.png!small)

#### **UDP探测时填充值修改** <a id="h3-11"></a>

osscan2.cc

static u8 patternbyte = 0x43; /\* character 'C' /

替换为 static u8 patternbyte = 0x46; / character 'F' \*/

重新编译后再去扫描，内容已经换了，长度应该也是可以调整。

u8 packet\[328\]; /\* 20 IP hdr + 8 UDP hdr + 300 data \*/

这里还没测试，感兴趣可以自己去定义，看会不会有什么问题。

![](https://image.3001.net/images/20200916/1600245630_5f61cf7e4edc468c9f32c.png!small)

### **0x05 nmap编译安装** <a id="h2-5"></a>

nmap编译时可能会遇到如下错误，几乎都是缺少特定的库导致的，

所以编译安装时需要安装以下库。

编译环境是基于Ubuntu的，其他环境库的名字可能不同，遇到编译报错可百度找对应解决方法即可。

```text
apt install flex bison libssl-dev./configure --without-zenmapmake && make install
```

### **0x06 总结** <a id="h2-6"></a>

#### **可修改文件及修改处** <a id="h3-12"></a>

#### **6.1、修改namp默认win窗口值。** <a id="h3-13"></a>

tcpip.cc

tcp-&gt;th\_win = hosts\(1-65535\)

#### **6.2、修改nmap-service-probes文件中关键词** <a id="h3-14"></a>

nmap，nm@nm，nm2@nm2，nm@p，nm，0PT10NS sip

这些值酌情替换。

#### **6.3、修改脚本中的值** <a id="h3-15"></a>

* nselib/http.lua

USERAGENT = stdnse.getscript\_args\('http.useragent'\)

* nselib/mssql.lua

搜索Nmap NSE然后替换

* nselib/sip.lua

搜索Nmap NSE然后替换

* scripts/http-sql-injection.nse

搜索sqlspider然后替换

* scripts/ssl-heartbleed.nse

搜索Nmap ssl-heartbleed替换

* nselib/rdp.lua

local cookie = "mstshash=nmap"

#### **6.4、修改使用-O参数发包填充内容** <a id="h3-16"></a>

osscan2.cc

static u8 patternbyte = 0x43; /\* character 'C' /

替换为 static u8 patternbyte = 0x46; / character 'F' \*/



