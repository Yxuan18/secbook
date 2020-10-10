# 识别主机指纹

## 1、

Nmap维护一个nmap-os-db数据库，存储了上千种操作系统信息，简单一点来说,Nmap通过TCP/IP协议栈的指纹信息来识别目标主机的操作系统信息，这主要是利用了RFC标准中，没有强制规范了TCP/IP的某些实现,于是不同的系统中TCP/IP的实现方案可能都有其特定的方式，这些细节上的差异，给nmap识别操作系统信息提供了方案，具体一点说,Nmap分别挑选一个close和open的端口，分别发送给一个经过精心设计的TCP/UDP数据包，当然这个数据包也可能是ICMP数据包。然后根据收到返回报文,生成一份系统指纹。通过对比检测生成的指纹和nmap-os-db数据库中的指纹，来查找匹配的系统。最坏的情况下，没有办法匹配的时候,则用概率的形式枚举出所有可能的信息。

所谓的指纹，即由特定的回复包提取出的数据特征

## 2、

Nmap-os-db在kali中路径如下

![v2-ecb242e1ab4f16f38c838eef12067aed\_hd.w](https://pic4.zhimg.com/80/v2-ecb242e1ab4f16f38c838eef12067aed_hd.webp)

我把他下在到win上方便查看

![v2-852c520bc8e56e7f1682124b195e838b\_hd.w](https://pic4.zhimg.com/80/v2-852c520bc8e56e7f1682124b195e838b_hd.webp)

这是指纹库的版本

以这条为例

![image.png](https://image.3001.net/images/20190517/1558062939_5cde275bd8727.png!small)

最前面几行为注释行，说明此指纹对应的操作系统与版本。

Fingerprint关键字定义一个新的指纹，紧随其后的是指纹名字。

Class行用于指定该指纹所属的类别，依次指定该系统的vendor（生产厂家）,OS family（系统类别）,OS generation（第几代操作系统）,and device type（设备类型）。

接下来是CPE行，此行非常重要，使用CPE（CommonPlatformEnumeration，通用平台枚举）格式描述该系统的信息。以标准的CPE格式来描述操作系统类型，便于Nmap与外界信息的交换，比如可以很快从网上开源数据库查找到CPE描述的操作系统具体信息。

此处作为指纹描述字段的CPE格式如下：

```text
cpe:/<part>:<vendor>:<product>:<version>:<update>:<edition>:<language>
```

接下来从SEQ到IE的13行都是具体指纹数据描述行，在对比指纹时，就是对比这13行里面的具体数据，如果匹配则目标机为指纹所描述的系统类型。

SEQ描述顺序产生方式；OPS描述TCP包中可选字段的值；WIN描述TCP包的初始窗口大小；ECN（ExplicitCongestionNotification）描述TCP明确指定拥塞通知时的特征；T1-T7描述TCP回复包的字段特征；U1描述向关闭的UDP发包产生的回复的特征；IE描述向目标机发送ICMP包产生的特征。

## 3、

在系统探测过程中，会执行五种不同的测试，每种测试由一个或者多个数据包组成，目标系统对每个数据包作出的响应有助于确定操作系统的类型。

五种不同的测试是：

1. sequencegeneration
2. ICMPecho
3. tcp explicit congestion notification
4. TCP
5. UDP

分别看看

序列生成\(sequencegeneration\)：

序列生成测试由六个数据包组成，这六个包是每隔100 毫秒分开发送的，且都是TCP SYN 包。每个TCP SYN 包的结果将有助于NMAP 确定操作系统的类型。

ICMP回显（ICMPecho）：

两个有着不同设置的ICMP请求包被送到目标系统，由此产生的反应将有助于实现验证操作系统类型。

TCP显式拥塞通知（explicitcongestion notification）：

当生成许多包通过路由器时会导致其负载变大，这称之为拥塞。其结果就是系统会变慢以降低拥堵，以便路由器不会发生丢包。这个包仅为了得到目标系统的响应而发送。因为不同的操作系统以不同的方式处理这个包，所以返回的特定值可以用来判断操作系统。

TCP：在这个测试中会发送六个数据包。一些带有特定的包设置的包被发送用来到打开的或关闭的端口。结果也将会因为操作系统的不同而不同。

所有TCP 包都是以如下不同的标志被发送：

无标志

SYN、FIN、URG和PSH

ACK

SYN

ACK

FIN、PSH和URG

UDP：这个测试由一个被发送给一个关闭的端口的数据包组成。如果目标系统上的这个端口是关闭的，而且返回一条ICMP 端口不可达的信息，那么就说明没有防火墙。

## 4、

以kali为例，如果关闭全部端口，则会显示

![v2-770916837d72a466e053616d7ceba096\_hd.w](https://pic4.zhimg.com/80/v2-770916837d72a466e053616d7ceba096_hd.webp)

开放一个80端口，此时就可以检测出这是linux系统

![v2-79ea2ab98876bc7fbf8ec12945995f10\_hd.w](https://pic2.zhimg.com/80/v2-79ea2ab98876bc7fbf8ec12945995f10_hd.webp)

![v2-a9b9d9bd37f3d11fd08c94e5b4c9a197\_hd.w](https://pic1.zhimg.com/80/v2-a9b9d9bd37f3d11fd08c94e5b4c9a197_hd.webp)

## 5、

接下来通过抓包分析

144为被扫描的机器，138为运行nmap的机器

![image.png](https://image.3001.net/images/20190517/1558062981_5cde27856f6ce.png!small)

我在前面提到，在kali上开发的唯一端口是80，所以在wireshark可以看到这一系列包是在发往80端口的

1. Sequence generation \(SEQ, OPS, WIN, and T1\)

会发送一系列共6个tcp探测来生成4个响应行，每一个都是tcpsyn数据包，连接到远程机器上检测到的开放的端口。

这些数据包的序列（sequence）和确认号（acknowledgementnumbers）是随机的，tcp选项和tcp窗口字段值也是不同的。

具体而言如下所示：

```text
Packet #1: window scale (10), NOP, MSS (1460),timestamp (TSval: 0xFFFFFFFF; TSecr: 0), SACK permitted. The windowfield is 1.
```

如2006所示

![v2-f7941270f56001390282b912b56a397c\_hd.w](https://pic3.zhimg.com/80/v2-f7941270f56001390282b912b56a397c_hd.webp)

```text
Packet #2: MSS (1400), window scale (0), SACKpermitted, timestamp (TSval: 0xFFFFFFFF; TSecr: 0), EOL. The windowfield is 63.
```

如2009所示

![v2-8f8a6173983af30c4d59d967145593c6\_hd.w](https://pic4.zhimg.com/80/v2-8f8a6173983af30c4d59d967145593c6_hd.webp)

```text
Packet #3: Timestamp (TSval: 0xFFFFFFFF; TSecr:0), NOP, NOP, window scale (5), NOP, MSS (640). The window field is4.
```

如2012所示

![v2-b90c9b544a925ffb5d49693f1e4aa957\_hd.w](https://pic4.zhimg.com/80/v2-b90c9b544a925ffb5d49693f1e4aa957_hd.webp)

```text
Packet #4: SACK permitted, Timestamp (TSval:0xFFFFFFFF; TSecr: 0), window scale (10), EOL. The window field is 4.
```

如2015所示

![v2-a786a9d734bdc823e7bb3b2e6a55ab84\_hd.w](https://pic3.zhimg.com/80/v2-a786a9d734bdc823e7bb3b2e6a55ab84_hd.webp)

```text
Packet #5: MSS (536), SACK permitted, Timestamp(TSval: 0xFFFFFFFF; TSecr: 0), window scale (10), EOL. The windowfield is 16.
```

如2018所示

![image.png](https://image.3001.net/images/20190517/1558063117_5cde280d852d4.png!small)

```text
Packet #6: MSS (265), SACK permitted, Timestamp(TSval: 0xFFFFFFFF; TSecr: 0). The window field is 512.
```

如2021所示

![v2-11f73582b8d7e78efb15edcd63daa5c7\_hd.w](https://pic4.zhimg.com/80/v2-11f73582b8d7e78efb15edcd63daa5c7_hd.webp)

![v2-877a1007da6b8d88d8ca57d75985d650\_hd.w](https://pic3.zhimg.com/80/v2-877a1007da6b8d88d8ca57d75985d650_hd.webp)

上图中2006-2007是一对syn,及对应返回的synack;

2006-2007,2009-2010,2012-2013,2015-2016,2018-2019,2021-2022一共6对

这些测试的结果包括四个结果类别行。

第一个SEQ包含基于探测包的序列分析的结果。这些测试结果是GCD，SP，ISR，TI，II，TS和SS。

SEQ测试将六个TCPSYN数据包发送到目标机器的开放端口，并收回SYN/ ACK数据包。这些SYN /ACK分组中的每一个包含32位初始序列号（ISN）。GCD,SP,ISR的计算比较麻烦。GCD根据ISN计算。ISR,SP都根据GCD计算。

下面的截图是6个tcpsyn包中的ISN

![v2-bf88aec1d6f9bb158a91239100a88bd8\_hd.w](https://pic1.zhimg.com/80/v2-bf88aec1d6f9bb158a91239100a88bd8_hd.webp)

![v2-136a6f3027120bc58991e3335381c0af\_hd.w](https://pic3.zhimg.com/80/v2-136a6f3027120bc58991e3335381c0af_hd.webp)

![v2-c2425a73decb0ff7d5078cb3ebf8175b\_hd.w](https://pic4.zhimg.com/80/v2-c2425a73decb0ff7d5078cb3ebf8175b_hd.webp)

![v2-3b740081c92e98c5bfac0ec302a66bf7\_hd.w](https://pic3.zhimg.com/80/v2-3b740081c92e98c5bfac0ec302a66bf7_hd.webp)

![v2-3c90ecb64cd85c1d83c6d6c40697211d\_hd.w](https://pic3.zhimg.com/80/v2-3c90ecb64cd85c1d83c6d6c40697211d_hd.webp)

TI会检查响应的IP头ID字段，必须至少收到三个响应才能包含测试，如果ID字段值都是0的话，则为Z在2007,2010,2013,2016,2019,2022数据包中的IP头部ID字段均为0

![v2-24988a42430bfc941b9c8f523d39400d\_hd.w](https://pic1.zhimg.com/80/v2-24988a42430bfc941b9c8f523d39400d_hd.webp)

所以在TI的值为Z

TS是根据SEQ探测的响应中的TCP时间戳选项，它检查TSval（选项的前四个字节）

如果时间戳选项值不为0，还需计算，比较麻烦，根据计算结果再赋TS值为1或7或8

从数据包中，以2007为例，可以知道TS值为1或7或8

![v2-cc04829249b59e377fd360122d5ba19c\_hd.w](https://pic1.zhimg.com/80/v2-cc04829249b59e377fd360122d5ba19c_hd.webp)

下一行OPS包含为每个探测器接收的TCPoption（测试名称为O1到O6）。

按照顺序来，即2007为O1,2010位O2...

以2007为例来分析

![v2-5895daf4a4449f7eb950ab00e2752fd2\_hd.w](https://pic4.zhimg.com/80/v2-5895daf4a4449f7eb950ab00e2752fd2_hd.webp)

它对应的字符串是M5B4ST11NW7:

M代表Maximumsegment size,1460的16进制为5B4;

S代表SackPermitted

T代表Timestamp，如果TSval,TSecr都不是0，则为11

N代表NOP

W代表Windowscale,大小为7

O2-O6以此类推

WIN行包含response的windowsize（名为W1到W6）。

以2013为例

![v2-a488b275324d60f19ace6561c4d33020\_hd.w](https://pic3.zhimg.com/80/v2-a488b275324d60f19ace6561c4d33020_hd.webp)

Windowsize为28960

与这些探测器相关的最后一行T1包含packet＃1的各种测试值。这些结果用于R，DF，T，TG，W，S，A，F，O，RD和Q测试。这些测试仅针对第一个探针报告，因为它们对于每个探针几乎总是相同的

R表示目标是否有响应，有响应则为Y

DF表示禁止路由器分段数据包的位是否置位，若置位则为Y,从下图可以看出已置位

![image.png](https://image.3001.net/images/20190517/1558063159_5cde2837ac423.png!small)

T表示初始TTL,下图可以看到T应为39

![image.png](https://image.3001.net/images/20190517/1558063171_5cde284337081.png!small)

TG为猜测的初始TTL值，如果发现实际TTL值，则不会打印该字段

S检查TCP报头中的32位序列号字段，与引发响应的探测中的TCP确认号进行比较。然后它记录适当的值。下图可以看到sequencenumber为0，所以S的值为Z

![image.png](https://image.3001.net/images/20190517/1558063182_5cde284e21468.png!small)

A测试响应中的确认号acknowledgementnumber与相应探测中的序列号的比较

下图中可以看到2017的acknowledgementnumber为1，2016中的sequencenumber 为0，0+1=1，即2017的acknowledgementnumber等于2016的sequencenumber+1

所以A的值为S+

F记录响应中的tcpflag

下图以2017为例，flags中A和S置位，所以F的值为AS

![v2-a435ef037f1478fa1aa2de4f1e690746\_hd.w](https://pic4.zhimg.com/80/v2-a435ef037f1478fa1aa2de4f1e690746_hd.webp)

RD是针对reset包的数据做校验和的结果，如果没数据或没校验或校验和无效，则为0

下图可以看出是没校验，RD值为0

![v2-f5883d542c6b04699219a3d563d5a403\_hd.w](https://pic3.zhimg.com/80/v2-f5883d542c6b04699219a3d563d5a403_hd.webp)

Q主要针对两处：一处是tcpheader的保留字段非0，如果出现则Q中记录“R”

![v2-6982f1234c38a2c50660c7e19ef565b0\_hd.w](https://pic3.zhimg.com/80/v2-6982f1234c38a2c50660c7e19ef565b0_hd.webp)

另一处是没设置URGflag时，存在非零的URG指针字段

上图可以看出都不存在，所以Q为空

