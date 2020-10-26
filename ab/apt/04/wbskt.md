# 利用WebSocket隧道的新型攻击活动披露

## 1、攻击技术分析

相关攻击第一阶段的恶意荷载主要利用DotNetToJScript技术，以JavaScript恶意脚本为载体，在内存中执行.NET Assembly形式的恶意功能组件，以无文件攻击的方式躲避安全软件的查杀。

第二阶段的后门程序会通过C&C下载各类攻击组件，其中核心的攻击动作是在失陷机器上利用WebSocket隧道建立反向的Socks5代理，操作各种后门和攻击组件进行渗透攻击。由于WebSocket隧道属于加密流量，部分流量安全设备很难识别异常情况。

![](../../../.gitbook/assets/image%20%28711%29.png)

## 2、核心加载程序

第一阶段的核心加载程序是DLL形式的恶意荷载，按功能特点我们将其命名为JsLoader。其主要功能是从C&C服务器下载加密的JS脚本文件在内存中执行。

| 名称 | MD5 | 类型 |
| :--- | :--- | :--- |
| JsLoader | 7680ea7601c90078f07f7b9be1944b3b | PE32（DLL） |

JsLoader是由Delphi语言编写，在其主要功能函数中，是通过与密钥“MF8Ya”进行异或运算解密全局数据来获取配置信息。配置信息为首次需要访问的URL和注册表路径等信息，主要信息如下：

a\)URL：

```text
https://v1ew0[.]net/MF8YaoBm1D1QTpk2TYkOTzb2pjvbRH0JLf0a2g1S/-1/1904/25cca85/resources/frAQBc8W
```

b\)注册表：

```text
HKEY_CURRENT_USER\Software\Gallery2.0
```

紧接着该程序会尝试加载amsi.dll，对AmsiScanBuffer函数进行inline hook处理，躲避反病毒软件的检测。

![](../../../.gitbook/assets/image%20%28696%29.png)

然后枚举获取HKCU\Software\Gallery2.0下的键值保存起来，使用事先保存在TLS中的回调函数中创建线程对数据进行解密，针对解密后的数据后使用jscript组件对其进行加载执行。其解密算法为跳过数据的前12个字节后，与密钥“MF8Ya”进行异或运算

![](../../../.gitbook/assets/image%20%28700%29.png)

![](../../../.gitbook/assets/image%20%28728%29.png)

![](../../../.gitbook/assets/image%20%28686%29.png)

在此之后，JsLoader会通过访问bing.com、baidu.com、google.com进行网络连通性测试，然后与最开始解密出的URL建立HTTP会话，并获取服务器响应数据，此段数据为服务器下发的JS恶意脚本，程序会对数据进行异或解密操作，并调用jscript组件对其进行加载执行。

相关的JS脚本是通过DotNetToJScript技术加载base64编码的.NET Assembly数据，将恶意组件反射加载到内存中执行。

![](../../../.gitbook/assets/image%20%28719%29.png)

![](../../../.gitbook/assets/image%20%28698%29.png)

![](../../../.gitbook/assets/image%20%28753%29.png)

JsLoader程序在与C&C服务器进行HTTP通信的过程中，还会有如下网络行为特点：

1.如果响应请求为HTTP 505，则该样本会创建另一个自身进程，并退出。

2.HTTP请求为GET请求，需要有9轮HTTP 200的会话响应，每一轮的请求参数有如下规律：

a\)第一轮：URL/true?x=5

b\)第二轮：URL/true?x=4,5

c\)第三轮：URL/true?x=3,4,5

d\)…

![](../../../.gitbook/assets/image%20%28745%29.png)

3.当响应状态不是HTTP 200时，则会将请求中的“true”替换为”false”，并再次向服务器发送请求，直到返回HTTP 200。

![](../../../.gitbook/assets/image%20%28709%29.png)

4.当9轮的HTTP 200会话完成后，则会将”true”替换为”mt10”，持续与服务器进行通信

![](../../../.gitbook/assets/image%20%28725%29.png)

## 3、恶意功能组件

在C&C下发的JavaScript脚本中，我们捕获到了三个不同功能的恶意组件，按功能特点我们分别将其命名为TunnelCore、FileRAT和ShellRAT，如下表所示：

| 名称 | 文件名 | 功能说明 |
| :--- | :--- | :--- |
| TunnelCore | ProxyModule.dll | 利用WebSocket隧道建立Socks5代理进行，为其他组件提供内网穿透通道 |
| FileRat | LiveFileManagerModule.dll | 本地HTTP服务器，监听本地12380端口，提供多种文件操作功能 |
| ShellRat | LiveConsoleModule.dll | 本地TCP服务器，监听本地12323端口，提供远程Shell功能 |

### **TunnelCore**

TunnelCore属于核心的代理隧道组件，通过传入指定的URL参数发起WebSocket连接。

![](../../../.gitbook/assets/image%20%28702%29.png)

利用WebSocket协议伪装成正常的Web请求，建立通信隧道。

![](../../../.gitbook/assets/image%20%28749%29.png)

紧接着利用WebSocket隧道建立Socks5代理实现内网穿透

![](../../../.gitbook/assets/image%20%28765%29.png)

### **FileRat**

FileRat样本会建立本地HTTP服务器，监听本地的12380端口，同时利用TunnelCore组件实现的内网端口转发，黑客可以远程访问该HTTP服务器以达到控制目标主机的目的。

![](../../../.gitbook/assets/image%20%28738%29.png)

![](../../../.gitbook/assets/image%20%28720%29.png)

其主要提供了如下功能：

| 访问路径 | 说明 |
| :--- | :--- |
| / | 发送404响应 |
| /api/ | 执行后续操作所需访问的父路径 |
| /api/getZipBundle | 将指定文件进行ZIP压缩 |
| /api/sendFile | 发送指定文件 |
| /api/getDriveInfo | 枚举获取驱动器信息 |
| /api/getDirectoryInfo | 获取指定目录文件信息 |
| /api/closeModule | 无 |
| /api/delete | 删除指定文件 |
| /api/getFileListing | 枚举驱动器，获取所有文件信息 |

### **ShellRAT**

ShellRAT属于远程Shell组件，会在本地监听TCP 12323端口，同时利用TunnelCore组件实现的内网端口转发，黑客可以远程Shell命令

![](../../../.gitbook/assets/image%20%28682%29.png)

![](../../../.gitbook/assets/image%20%28699%29.png)

在与cmd交互的过程中，其输入缓冲区的第一个字节为特殊功能号，具体如下：

| 特殊功能号 | 说明 |
| :--- | :--- |
| ‘\u0003’ | 结束远程Shell |
| ‘\u0004’ | 暂停远程Shell |
| ‘\u0010’ | 与远程Shell的交互，执行命令 |

其中与cmd进行交互中，会额外将命令缓存在@string所标识的文件流中。

![](../../../.gitbook/assets/image%20%28739%29.png)

采取的格式如下：

| 1byte | 62bytes | 4bytes | Buffer |
| :--- | :--- | :--- | :--- |
| '\u0010'功能号 | @string代表文件名 | 长度 | 命令 |

## 4、关联分析

核心加载程序JsLoader 的dll导出函数与duser.dll一致，是常用于DLL劫持类攻击的荷载。

![](../../../.gitbook/assets/image%20%28746%29.png)

响尾蛇组织历史上的多次攻击活动惯用credwiz.exe+duser.dll组合的白利用攻击手法，通过360安全大脑根据相关攻击模式对JsLoader的分析显示，相关样本与响尾蛇（SideWinder）组织的多次攻击活动存在关联。

## 附录 IOCs

```text
## URL
https://v1ew0[.]net/MF8YaoBm1D1QTpk2TYkOTzb2pjvbRH0JLf0a2g1S/-1/1904/25cca85/resources/frAQBc8W
https://ms10t[.]net/9SH3VxPLwcngKM2yO2gz2comZ9vd06g4w485XFLR/36847/1595/2a182725/resources/frAQBc8W

## 注册表
HKUR\Software\Gally2.0

## MD5
7680ea7601c90078f07f7b9be1944b3b
fb88fc5a6f4ada89b3fc3e3bbe6d532d
f5547a680651fc361089f3aea03a3cbd
e1a387dbaa1cb89b4218d301abe47308

## 同族样本
1388d61b4abc3734772e1dc5c279e5c3
dfc0a83e39268fca21d84f8ee2e9964e
```

