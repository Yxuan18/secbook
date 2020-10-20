# Network MapTools

##  一、使用详解

### 1、nmap介绍 <a id="h2-1"></a>

Nmap是一个开源，免费的网络探测工具，可以用来探测存活主机以及开放的端口等，支持windows，linux，mac等多种操作系统。

nmap下载官网https://nmap.org/download.html

### 2、nmap常用功能 <a id="h2-2"></a>

- 探测主机存活  
- 扫描端口  
- 探测主机操作系统信息  
- 检测漏洞

### 3、nmap实战 <a id="h2-3"></a>

nmap 常用的几个参数

```text
nmap -v ip 显示详细的扫描过程
nmap -p  ip  扫描指定端口
nmap -A ip  全面扫描操作系统
nmap -sP ip  进行ping扫描主机存活
nmap -Pn/-P0 ip 禁ping扫描
nmap -sS ip 进行tcp syn扫描 也叫半开放扫描
nmap -sT ip 进行tcp连接扫描 （准确性高）
```

端口状态信息

&gt; open ：端口开启  
&gt; closed ：端口关闭  
&gt; filtered ：端口被过滤，因为报文被防火墙拦截  
&gt; Unfiltered ：不确定端口是否开放 没有被过滤  
&gt; open\|filtered\(closed\|filtered\) ：不能确定端口是否开放（关闭）或者被过滤

### 4、nmap中文说明文档 <a id="h2-4"></a>

TARGET SPECIFICATION  目标说明

-iL 从主机地址列表文件中导入扫描地址  
-iR 随机选择目标进行扫描，unm hosts 表示数目，设置为0则无休止扫描  
--exclude 排除某个主机地址  
--excludefile 排除主机列表文件中的地址  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
HOST DISCOVERY 主机发现

-sL 列表扫描，仅将指定的目标IP列举出来，不进行主机发现  
-sn 和-sP一样，只利用ping扫描进行主机发现，不扫描目标主机的端口  
-Pn 将所有指定的主机视为已开启状态，跳过主机发现过程  
-PS  TCP SYN ping ，发送一个设置了SYN标志位的TCP报文，默认端口为80，也可指定端口  
-PA  TCP ACK ping，发送一个设置了ACK标志位的TCP报文，默认端口为80，也可指定带你看  
-PU  UDP ping ,发送一个空的UDP报文到指定端口，可穿透只过滤TCp的防火墙  
-P0 使用IP协议ping  
-PR 使用ARP ping  
-n/-R    -n不用域名解析，加速扫描，-R为目标IP做为反向解析域名，扫描较慢一些  
-dns-servers 自定义域名解析服务器地址  
-traceroute  目标主机路由追踪  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SCAN TECHNIQUER  端口扫描

nmap 将目标端口分为6种状态：  
open（开放的）  
closed（关闭的）  
filtered （被过滤的）  
unfiltered （未被过滤的）可访问但不确定开发情况  
open\|filtered （开放或者被过滤）无法确定端口是否开放的还是被过滤的  
closed\|filtered （关闭或者被过滤）无法确定端口是否关闭还是被过滤的  
nmap产生的结果基于目标机器的响应报文的，而这些主机肯能是不可信任的。会产生迷惑或者误导nmap的报文

-sS   TCP SYN 扫描，不进行ping 半开放扫描，速度快隐蔽性好（不完成TCP连接），能够明确区分端口状态  
-sT   TCP 连接扫描，容易产生记录，效率低  
-sA   TCP ACK扫描，只设置ACK标志位，区别被过滤与未被过滤的  
-sU   UDP 服务扫描，例如DNS/DHCP等，效率低  
-sN;-sF;-sX    TCP Null,Fin,Xmas扫描，从RFC挖掘的微妙方法来区分开放关闭端口  
-sl    利用僵尸主机上已知IP分段ID序列生成算法来探测目标上开放端口的信息，极端隐蔽  
-sO  IP协议扫描，可以确定目标主机支持哪些IP协议，例如TCP/ICMP等  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
PORT SPECIFICATION AND SCAN ORDER 端口说明和扫描顺序-p  指定扫描的端口，可以是单个端口，也可以是端口范围，可以指定UDP或TCP协议扫描特定端口  
-p&lt;name&gt; 指定扫描的协议，例如-p http 即可扫描http协议的端口状态  
--exclude-ports  排除指定端口不扫描  
-F 极速模式，仅扫描100个常用端口  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SERVICE/VERSION DETECTION  服务与版本探测。

Nmap-services  包含大量服务的数据库，Nmap通过查询该数据库可以报告哪些端口可能对应于扫描服务，但不一定正确  
-sV 进行服务版本探测  
--version-intensity&lt;level&gt; 设置版本扫描强度，范围为0-9，默认是7，强度越高，时间越长，服务越可能被正确识别  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SCRIPT SCAN  脚本扫描

允许用户自己编写的脚本来执行自动化的操作或者扩展nmap的功能，使用Lua脚本语言。  
-sC  使用默认类别的脚本进行扫描  
--script=&lt;Lua scripts&gt;  使用某个或某类脚本进行扫描，支持统配符描述  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
OS DETECTION 操作系统探测

用TCP/IP协议栈fingerprinting进行探测，Nmap发送一系列TCP和UDP报文到远程主机，检查响应的每一个比特。测试后Nmap吧结果和数据库中超过1500个已知的fingerprints比较，如匹配则输出结果

-O  启用操作系统探测  
-A 同时启用操作系统探测和服务版本探测  
--osscan-limit 针对指定的目标进行操作系统检测  
--osscan-guess  当nmap无法确定所检测的操作系统时，会尽可能地提供相近的匹配  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
TIMING AND PERFORMANCE  时间和性能  
Nmap开发的最高优先级是性能，但实际应用中很多因素会添加扫描时间，比如特定的扫描选项，防火墙配置以及版本扫描等

-T&lt;0-5&gt; 设置模板级数，在0-5中选择。T0、T1用于IDS规避，T2降低了扫描速度以使用更少的带宽和资源。默认为T3，未作任何优化。T4假设具有合适及可靠的网络从而加速扫描。T5假设具有特别快的网络或者愿为速度牺牲准确性  
-host-timeout&lt;time&gt; 放弃低速目标主机，时间单位为毫秒

#### 扫描漏洞 <a id="h3-1"></a>

#### nmap --script=vuln ip <a id="h3-2"></a>

3、提供暴力破解的方式，可对数据库、smb、snmp等进行简单密码的暴力猜解  
nmap --script=brute 192.168.88.131

4、利用FTP指定脚本对目标特定FTP协议进行密码爆破  
nmap --script=ftp-brute.nse 192.168.88.131

5、利用第三方的数据库或资源，例如进行whoise解析  
nmap --script=external 192.168.88.131

##  二、详解

###  1、功能点及原理

####  1、主机发现

概念：即用于发现目标主机是否在线（Alive，处于开启状态）

原理：

主机发现发现的原理与Ping命令类似，发送探测包到目标主机，如果收到回复，那么说明目标主机是开启的。Nmap支持十多种不同的主机探测方式，比如发送ICMP ECHO/TIMESTAMP/NETMASK报文、发送TCPSYN/ACK包、发送SCTP INIT/COOKIE-ECHO包，用户可以在不同的条件下灵活选用不同的方式来探测目标机。

主机发现基本原理：（以ICMP echo方式为例）

![](../../.gitbook/assets/image%20%28655%29.png)

Nmap的用户位于源端，IP地址192.168.0.5，向目标主机192.168.0.3发送ICMP Echo Request。如果该请求报文没有被防火墙拦截掉，那么目标机会回复ICMP Echo Reply包回来。以此来确定目标主机是否在线。

默认情况下，Nmap会发送四种不同类型的数据包来探测目标主机是否在线。

1. ICMP echo request
2. a TCP SYN packet to port 443
3. a TCP ACK packet to port 80
4. an ICMP timestamp request

依次发送四个报文探测目标机是否开启。只要收到其中一个包的回复，那就证明目标机开启。使用四种不同类型的数据包可以避免因防火墙或丢包造成的判断错误。

####  2、端口扫描

{% tabs %}
{% tab title="TCP SYN scanning" %}
这是Nmap默认的扫描方式，通常被称作半开放扫描（Half-open scanning）。该方式发送SYN到目标端口，如果收到SYN/ACK回复，那么判断端口是开放的；如果收到RST包，说明该端口是关闭的。如果没有收到回复，那么判断该端口被屏蔽（Filtered）。因为该方式仅发送SYN包对目标主机的特定端口，但不建立的完整的TCP连接，所以相对比较隐蔽，而且效率比较高，适用范围广。

TCP SYN探测到端口关闭：

![](../../.gitbook/assets/image%20%28650%29.png)

TCP SYN探测到端口开放：

![](../../.gitbook/assets/image%20%28647%29.png)
{% endtab %}

{% tab title="TCP connect scanning" %}
TCP connect方式使用系统网络API connect向目标主机的端口发起连接，如果无法连接，说明该端口关闭。该方式扫描速度比较慢，而且由于建立完整的TCP连接会在目标机上留下记录信息，不够隐蔽。所以，TCP connect是TCP SYN无法使用才考虑选择的方式。

TCP connect探测到端口关闭：

![](../../.gitbook/assets/image%20%28652%29.png)

TCP connect探测到端口开放：

![](../../.gitbook/assets/image%20%28649%29.png)
{% endtab %}

{% tab title="TCP ACK scanning" %}
向目标主机的端口发送ACK包，如果收到RST包，说明该端口没有被防火墙屏蔽；没有收到RST包，说明被屏蔽。该方式只能用于确定防火墙是否屏蔽某个端口，可以辅助TCP SYN的方式来判断目标主机防火墙的状况。

TCP ACK探测到端口被屏蔽：

![](../../.gitbook/assets/image%20%28653%29.png)

TCP ACK探测到端口未被屏蔽：

![](../../.gitbook/assets/image%20%28648%29.png)
{% endtab %}

{% tab title="TCP FIN/Xmas/NULL scanning" %}
这三种扫描方式被称为秘密扫描（Stealthy Scan），因为相对比较隐蔽。FIN扫描向目标主机的端口发送的TCP FIN包或Xmas tree包/Null包，如果收到对方RST回复包，那么说明该端口是关闭的；没有收到RST包说明端口可能是开放的或被屏蔽的（open\|filtered）。

其中Xmas tree包是指flags中FIN URG PUSH被置为1的TCP包；NULL包是指所有flags都为0的TCP包。

TCP FIN探测到主机端口是关闭的：

![](../../.gitbook/assets/image%20%28656%29.png)

TCP FIN探测到主机端口是开放或屏蔽的：

![](../../.gitbook/assets/image%20%28654%29.png)

\*\*\*\*
{% endtab %}

{% tab title="UDP scanning" %}
UDP扫描方式用于判断UDP端口的情况。向目标主机的UDP端口发送探测包，如果收到回复“ICMP port unreachable”就说明该端口是关闭的；如果没有收到回复，那说明UDP端口可能是开放的或屏蔽的。因此，通过反向排除法的方式来断定哪些UDP端口是可能出于开放状态。

UDP端口关闭：

![](../../.gitbook/assets/image%20%28651%29.png)

UDP端口开放或被屏蔽：

![](../../.gitbook/assets/image%20%28657%29.png)
{% endtab %}

{% tab title="其他方式" %}
 例如使用SCTP INIT/COOKIE-ECHO方式来探测SCTP的端口开放情况；使用IP protocol方式来探测目标主机支持的协议类型（TCP/UDP/ICMP/SCTP等等）；使用idle scan方式借助僵尸主机（zombie host，也被称为idle host，该主机处于空闲状态并且它的IPID方式为递增。详细实现原理参见：[http://nmap.org/book/idlescan.html](http://nmap.org/book/idlescan.html)）来扫描目标在主机，达到隐蔽自己的目的；或者使用FTP bounce scan，借助FTP允许的代理服务扫描其他的主机，同样达到隐藏自己的身份的目的。
{% endtab %}
{% endtabs %}

#### 3、版本侦测原理

简要的介绍版本的侦测原理。

版本侦测主要分为以下几个步骤：

1. 首先检查open与open\|filtered状态的端口是否在排除端口列表内。如果在排除列表，将该端口剔除。
2. 如果是TCP端口，尝试建立TCP连接。尝试等待片刻（通常6秒或更多，具体时间可以查询文件nmap-services-probes中Probe TCP NULL q\|\|对应的totalwaitms）。通常在等待时间内，会接收到目标机发送的“WelcomeBanner”信息。nmap将接收到的Banner与nmap-services-probes中NULL probe中的签名进行对比。查找对应应用程序的名字与版本信息。
3. 如果通过“Welcome Banner”无法确定应用程序版本，那么nmap再尝试发送其他的探测包（即从nmap-services-probes中挑选合适的probe），将probe得到回复包与数据库中的签名进行对比。如果反复探测都无法得出具体应用，那么打印出应用返回报文，让用户自行进一步判定。
4. 如果是UDP端口，那么直接使用nmap-services-probes中探测包进行探测匹配。根据结果对比分析出UDP应用服务类型。
5. 如果探测到应用程序是SSL，那么调用openSSL进一步的侦查运行在SSL之上的具体的应用类型。
6. 如果探测到应用程序是SunRPC，那么调用brute-force RPC grinder进一步探测具体服务。

#### 4、OS侦测

Nmap使用TCP/IP协议栈指纹来识别不同的操作系统和设备。在RFC规范中，有些地方对TCP/IP的实现并没有强制规定，由此不同的TCP/IP方案中可能都有自己的特定方式。Nmap主要是根据这些细节上的差异来判断操作系统的类型的。

具体实现方式如下：

1. Nmap内部包含了2600多已知系统的指纹特征（在文件nmap-os-db文件中）。将此指纹数据库作为进行指纹对比的样本库。
2. 分别挑选一个open和closed的端口，向其发送经过精心设计的TCP/UDP/ICMP数据包，根据返回的数据包生成一份系统指纹。
3. 将探测生成的指纹与nmap-os-db中指纹进行对比，查找匹配的系统。如果无法匹配，以概率形式列举出可能的系统。









